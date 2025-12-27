# EveryTime Crawling
## 코드
```python
import logging
import math
import re
from collections import defaultdict

from fastapi import Request
from playwright.sync_api import Page, TimeoutError

from app.services.page_pool import PlaywrightPagePool

logger = logging.getLogger(__name__)

def crawl_schedule(url: str, request: Request, pool: PlaywrightPagePool):
    page: Page = pool.get_page()

    try:
        # 페이지 이동
        page.goto(url, timeout=15_000)

        # tablebody 로드 대기
        page.wait_for_selector(".tablebody", timeout=10_000)

        # tablehead (요일)
        days = [
            td.inner_text().strip()
            for td in page.query_selector_all(".tablehead td")
            if td.inner_text().strip()
        ]

        if not days:
            return {
                "code": "CRAWLING-003",
                "message": "공개되지 않은 시간표입니다.",
                "is_success": False
            }

        subjects = page.query_selector_all(".tablebody .subject")
        schedules = defaultdict(list)

        for subject in subjects:
            style = subject.get_attribute("style") or ""

            height_match = re.search(r'height:\s*(\d+)px', style)
            top_match = re.search(r'top:\s*(\d+)px', style)

            if not height_match or not top_match:
                continue

            height = int(height_match.group(1))
            top = int(top_match.group(1))

            # 시작 시간
            start_total_minutes = ((top - 450) // 25) * 30
            start_hour = 9 + (start_total_minutes // 60)
            start_minute = start_total_minutes % 60

            # 종료 시간
            duration_total_minutes = math.ceil((height - 1) / 25) * 30
            end_total_minutes = start_total_minutes + duration_total_minutes
            end_hour = 9 + (end_total_minutes // 60)
            end_minute = end_total_minutes % 60

            start_time = f"{start_hour:02}:{start_minute:02}"
            end_time = f"{end_hour:02}:{end_minute:02}"

            # 요일 계산
            parent_td = subject.locator("xpath=ancestor::td").element_handle()
            tr = parent_td.query_selector("xpath=ancestor::tr")
            tds = tr.query_selector_all("td")
            td_index = tds.index(parent_td)

            day = days[td_index] if td_index < len(days) else "알 수 없음"
            schedules[day].append((start_time, end_time))

        # 시간 병합
        final_schedules = []
        for day, times in schedules.items():
            times.sort()
            merged_times = set()

            for start, end in times:
                sh, sm = map(int, start.split(":"))
                eh, em = map(int, end.split(":"))

                ch, cm = sh, sm
                while (ch, cm) < (eh, em):
                    merged_times.add(f"{ch:02}:{cm:02}")
                    cm += 30
                    if cm >= 60:
                        ch += 1
                        cm = 0

            final_schedules.append({
                "time_point": day,
                "times": sorted(merged_times)
            })

        return {
            "code": "200",
            "message": "유저 고정 스케줄 조회에 성공했습니다.",
            "payload": {
                "schedules": final_schedules
            },
            "is_success": True
        }

    except TimeoutError:
        logger.warning("페이지 로딩 타임아웃")
        raise

    except Exception as e:
        logger.error(f"크롤링 중 예외 발생: {e}", exc_info=True)
        raise

    finally:
        logger.info("Page 반납")
        pool.release_page(page)
```
selenium에 비해 처리속도 2배 향상