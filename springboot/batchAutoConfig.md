# SrpingBoot 4.x Batch Auto Configuration
```java
/*
 * Copyright 2012-present the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.boot.batch.autoconfigure;

import org.jspecify.annotations.Nullable;

import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.support.DefaultBatchConfiguration;
import org.springframework.batch.core.converter.JobParametersConverter;
import org.springframework.batch.core.launch.JobOperator;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.task.TaskExecutor;

/**
 * {@link EnableAutoConfiguration Auto-configuration} for Spring Batch using an in-memory
 * store.
 *
 * @author Stephane Nicoll
 * @since 4.0.0
 */
@AutoConfiguration
@ConditionalOnClass(JobOperator.class)
@ConditionalOnMissingBean(value = DefaultBatchConfiguration.class, annotation = EnableBatchProcessing.class)
@EnableConfigurationProperties(BatchProperties.class)
public final class BatchAutoConfiguration {

    @Configuration(proxyBeanMethods = false)
    static class SpringBootBatchDefaultConfiguration extends DefaultBatchConfiguration {

        private final @Nullable TaskExecutor taskExecutor;

        private final @Nullable JobParametersConverter jobParametersConverter;

        SpringBootBatchDefaultConfiguration(@BatchTaskExecutor ObjectProvider<TaskExecutor> batchTaskExecutor,
                                            ObjectProvider<JobParametersConverter> jobParametersConverter) {
            this.taskExecutor = batchTaskExecutor.getIfAvailable();
            this.jobParametersConverter = jobParametersConverter.getIfAvailable();
        }

        @Override
        @Deprecated(since = "4.0.0", forRemoval = true)
        @SuppressWarnings("removal")
        protected JobParametersConverter getJobParametersConverter() {
            return (this.jobParametersConverter != null) ? this.jobParametersConverter
                    : super.getJobParametersConverter();
        }

        @Override
        protected TaskExecutor getTaskExecutor() {
            return (this.taskExecutor != null) ? this.taskExecutor : super.getTaskExecutor();
        }

    }

}
```
현재 Spring Boot 4.x의 BatchAutoConfiguration 에서는 DataSource를 등록할 수 없는 구조로 되어 있음
이는 Spring Batch 5.x에서 기본적으로 In-Memory Job Repository를 사용하도록 변경되었기 때문

commit comment를 확인해 보면 다음과 같음
> This commit moves the existing JDBC-based Spring Batch infrastructure
> to a new 'spring-boot-batch-jdbc' module, while the existing module
> only offers in-memory (aka resourceless) support.
> The commit also updates the reference guide to provide some more
> information about what's available and how to use it.

따라서 JDBC 기반을 사용하려면 Spring-boot-batch-jdbc 모듈을 별도로 추가해야 함
https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-batch-jdbc/4.0.0-RC2