# Bean 주입 심화
## 코드
```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
        implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {

    ...

    @Nullable
    public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
            @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

        InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
        try {
            Object shortcut = descriptor.resolveShortcut(this);
            if (shortcut != null) {
                return shortcut;
            }

            Class<?> type = descriptor.getDependencyType();
            Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
            if (value != null) {
                if (value instanceof String strValue) {
                    String resolvedValue = resolveEmbeddedValue(strValue);
                    BeanDefinition bd = (beanName != null && containsBean(beanName) ?
                            getMergedBeanDefinition(beanName) : null);
                    value = evaluateBeanDefinitionString(resolvedValue, bd);
                }
                TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
                try {
                    return converter.convertIfNecessary(value, type, descriptor.getTypeDescriptor());
                }
                catch (UnsupportedOperationException ex) {
                    // A custom TypeConverter which does not support TypeDescriptor resolution...
                    return (descriptor.getField() != null ?
                            converter.convertIfNecessary(value, type, descriptor.getField()) :
                            converter.convertIfNecessary(value, type, descriptor.getMethodParameter()));
                }
            }

            Object multipleBeans = resolveMultipleBeans(descriptor, beanName, autowiredBeanNames, typeConverter);
            if (multipleBeans != null) {
                return multipleBeans;
            }

            Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
            if (matchingBeans.isEmpty()) {
                if (isRequired(descriptor)) {
                    raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
                }
                return null;
            }

            String autowiredBeanName;
            Object instanceCandidate;

            if (matchingBeans.size() > 1) {
                autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
                if (autowiredBeanName == null) {
                    if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
                        return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
                    }
                    else {
                        // In case of an optional Collection/Map, silently ignore a non-unique case:
                        // possibly it was meant to be an empty collection of multiple regular beans
                        // (before 4.3 in particular when we didn't even look for collection beans).
                        return null;
                    }
                }
                instanceCandidate = matchingBeans.get(autowiredBeanName);
            }
            else {
                // We have exactly one match.
                Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
                autowiredBeanName = entry.getKey();
                instanceCandidate = entry.getValue();
            }

            if (autowiredBeanNames != null) {
                autowiredBeanNames.add(autowiredBeanName);
            }
            if (instanceCandidate instanceof Class) {
                instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
            }
            Object result = instanceCandidate;
            if (result instanceof NullBean) {
                if (isRequired(descriptor)) {
                    raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
                }
                result = null;
            }
            if (!ClassUtils.isAssignableValue(type, result)) {
                throw new BeanNotOfRequiredTypeException(autowiredBeanName, type, instanceCandidate.getClass());
            }
            return result;
        }
        finally {
            ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
        }
    }

    ...
}
```

Bean 이 존재하지 않는 경우 Runtime에서 `NullPointerException`이 발생
그런 이유로 @Autowired를 활용한 Field Injection는 deprecated 되고,
@RequiredArgsConstructor를 통한 Constructor Injection이 권장
> Field Injection의 경우에는 null이 될 것이고,
> Constructor Injection의 경우에는 final 접근자로 인해서 bean Injection 시점(Spring context loading 시)에 오류 발생

```java
Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
if (matchingBeans.isEmpty()) {
    if (isRequired(descriptor)) {
        raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
    }
    return null;
```
findAutowireCandidates()에서 bean injection의 후보군을 추렸을 때,  
1개인 경우 바로 주입되고, 여러 개인 경우에는 다시 determineAutowireCandidate()를 실행하게 됨  
size()는 0이상의 자연수이기 때문에 음수는 없을 것이고, 0은 앞의 로직에서 빠졌고,  
2이상은 determinAutowireCandidate()를 타기 때문에,  
else가 1의 경우로 암묵적 처리됨

```java
String autowiredBeanName;
Object instanceCandidate;

if (matchingBeans.size() > 1) {
    autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
    if (autowiredBeanName == null) {
        if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
            return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
        }
        else {
            // In case of an optional Collection/Map, silently ignore a non-unique case:
            // possibly it was meant to be an empty collection of multiple regular beans
            // (before 4.3 in particular when we didn't even look for collection beans).
            return null;
        }
    }
    instanceCandidate = matchingBeans.get(autowiredBeanName);
}
else {
    // We have exactly one match.
    Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
    autowiredBeanName = entry.getKey();
    instanceCandidate = entry.getValue();
}
```

findAutowireCandidates()를 살펴보면 Candidate라는 메서드 네이밍처럼,  
주입할 후보 대상을 찾음   
예전에는 명시적으로 bean injection을 타입으로 할지, 이름으로 할지 선택해야 했다면,  
요즘에는 타입이든 이름이든 모두 candidate로 만들어두고, 그중에서 가장 적합한 bean을 선택하게 되는 것으로 판단

```java
protected Map<String, Object> findAutowireCandidates(
        @Nullable String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {

    String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
            this, requiredType, true, descriptor.isEager());
    Map<String, Object> result = CollectionUtils.newLinkedHashMap(candidateNames.length);
    for (Map.Entry<Class<?>, Object> classObjectEntry : this.resolvableDependencies.entrySet()) {
        Class<?> autowiringType = classObjectEntry.getKey();
        if (autowiringType.isAssignableFrom(requiredType)) {
            Object autowiringValue = classObjectEntry.getValue();
            autowiringValue = AutowireUtils.resolveAutowiringValue(autowiringValue, requiredType);
            if (requiredType.isInstance(autowiringValue)) {
                result.put(ObjectUtils.identityToString(autowiringValue), autowiringValue);
                break;
            }
        }
    }
    for (String candidate : candidateNames) {
        if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, descriptor)) {
            addCandidateEntry(result, candidate, descriptor, requiredType);
        }
    }
    if (result.isEmpty()) {
        boolean multiple = indicatesMultipleBeans(requiredType);
        // Consider fallback matches if the first pass failed to find anything...
        DependencyDescriptor fallbackDescriptor = descriptor.forFallbackMatch();
        for (String candidate : candidateNames) {
            if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, fallbackDescriptor) &&
                    (!multiple || getAutowireCandidateResolver().hasQualifier(descriptor))) {
                addCandidateEntry(result, candidate, descriptor, requiredType);
            }
        }
        if (result.isEmpty() && !multiple) {
            // Consider self references as a final pass...
            // but in the case of a dependency collection, not the very same bean itself.
            for (String candidate : candidateNames) {
                if (isSelfReference(beanName, candidate) &&
                        (!(descriptor instanceof MultiElementDescriptor) || !beanName.equals(candidate)) &&
                        isAutowireCandidate(candidate, fallbackDescriptor)) {
                    addCandidateEntry(result, candidate, descriptor, requiredType);
                }
            }
        }
    }
    return result;
}
```
```java
@Nullable
protected String determineAutowireCandidate(Map<String, Object> candidates, DependencyDescriptor descriptor) {
    Class<?> requiredType = descriptor.getDependencyType();
    String primaryCandidate = determinePrimaryCandidate(candidates, requiredType);
    if (primaryCandidate != null) {
        return primaryCandidate;
    }
    String priorityCandidate = determineHighestPriorityCandidate(candidates, requiredType);
    if (priorityCandidate != null) {
        return priorityCandidate;
    }
    // Fallback
    for (Map.Entry<String, Object> entry : candidates.entrySet()) {
        String candidateName = entry.getKey();
        Object beanInstance = entry.getValue();
        if ((beanInstance != null && this.resolvableDependencies.containsValue(beanInstance)) ||
                matchesBeanName(candidateName, descriptor.getDependencyName())) {
            return candidateName;
        }
    }
    return null;
}
```
위 코드 `determineAutowireCandidate()` 에서 @Primary 어노테이션이 있으면 선주입
@Priority 어노테이션이 있으면 그 다음으로 선주입
둘 다 없으면 bean 이름과 의존성 이름이 일치하는 bean을 주입

@Primary가 있고 Multiple Bean Candidate가 있는 경우 무조건 @Primary가 가장 우선되어 적용
누군가 해당 Type의 Bean을 @Primary로 지정해두었다면,
다른 Bean이 여러 개 있더라도 @Primary가 붙은 Bean이 주입되는 목적과 다른 결과 발생할 가능성이 있음
테스트나 spring context loading 에서 bean이 여러 개라 ambiguous 하다는 오류가 발생했을 때 @Primary를 남발하면 안됨
  
findAutowireCandidates()를 실행하기전 코드블록에서 다음을 확인
```java
Class<?> type = descriptor.getDependencyType();
Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
if (value != null) {
    if (value instanceof String strValue) {
        String resolvedValue = resolveEmbeddedValue(strValue);
        BeanDefinition bd = (beanName != null && containsBean(beanName) ?
                getMergedBeanDefinition(beanName) : null);
        value = evaluateBeanDefinitionString(resolvedValue, bd);
    }
    TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
    try {
        return converter.convertIfNecessary(value, type, descriptor.getTypeDescriptor());
    }
    catch (UnsupportedOperationException ex) {
        // A custom TypeConverter which does not support TypeDescriptor resolution...
        return (descriptor.getField() != null ?
                converter.convertIfNecessary(value, type, descriptor.getField()) :
                converter.convertIfNecessary(value, type, descriptor.getMethodParameter()));
    }
}
```
AutowireCandidateResolver.getSuggestedValue()는 default로 null을 반환하고 있으나, interface로 정의되어 있음  
구현체를 찾아보면 QualifierAnnotationAutowireCandidateResolver.getSuggestedValue()를 사용

```java
/**
 * Determine whether the given dependency declares a value annotation.
 * @see Value
 */
@Override
@Nullable
public Object getSuggestedValue(DependencyDescriptor descriptor) {
    Object value = findValue(descriptor.getAnnotations());
    if (value == null) {
        MethodParameter methodParam = descriptor.getMethodParameter();
        if (methodParam != null) {
            value = findValue(methodParam.getMethodAnnotations());
        }
    }
    return value;
}

/**
 * Determine a suggested value from any of the given candidate annotations.
 */
@Nullable
protected Object findValue(Annotation[] annotationsToSearch) {
    if (annotationsToSearch.length > 0) {   // qualifier annotations have to be local
        AnnotationAttributes attr = AnnotatedElementUtils.getMergedAnnotationAttributes(
                AnnotatedElementUtils.forAnnotations(annotationsToSearch), this.valueAnnotationType);
        if (attr != null) {
            return extractValue(attr);
        }
    }
    return null;
}

/**
 * Extract the value attribute from the given annotation.
 * @since 4.3
 */
protected Object extractValue(AnnotationAttributes attr) {
    Object value = attr.get(AnnotationUtils.VALUE);
    if (value == null) {
        throw new IllegalStateException("Value annotation must have a value attribute");
    }
    return value;
}
```
@Qualifier 어노테이션 사용 시, Multiple Bean 검색하기 전에 있는지 없는지 찾아보고 Bean Injection을 시키고 없으면 오류 발생
@Qualifier > @Primary 순서로 우선순위가 적용됨