# -*- coding: utf-8 -*-
import streamlit as st
import requests
from bs4 import BeautifulSoup
from datetime import datetime

# --- 페이지 설정 ---
st.set_page_config(
    page_title="오늘의 날씨 알리미",
    page_icon="🌤️"
)

# --- 세션 상태(Session State) 초기화 ---
# 'weather'라는 키가 세션 상태에 없으면 None으로 초기화합니다.
# 세션 상태를 사용하면 앱이 재실행되어도 이전 데이터를 잠시 보관할 수 있습니다.
if 'weather' not in st.session_state:
    st.session_state['weather'] = None

# --- 핵심 기능: 날씨 정보 가져오기 ---
def fetch_weather():
    """
    웨더아이 사이트에서 서울 날씨 정보를 스크래핑하는 함수
    """
    try:
        # 목표 웹사이트 URL
        url = "https://www.weatheri.co.kr/forecast/forecast01.php"

        # 웹사이트에 GET 요청 보내기
        response = requests.get(url)
        response.raise_for_status() # 요청이 실패하면 오류 발생

        # HTML 파싱(Parsing)
        soup = BeautifulSoup(response.text, 'html.parser')

        # CSS 선택자(Selector)를 이용해 원하는 정보 찾기
        # 웹사이트 구조가 바뀌면 이 부분은 수정이 필요할 수 있습니다.
        # 개발자 도구(F12)를 사용해 구조를 확인하세요.
        location = soup.select_one('.w_map_list > .w_map_cont.on > a > span.loca').text.strip()
        weather_desc = soup.select_one('.w_map_list > .w_map_cont.on > .w_icon > span').text.strip()
        
        # 가져온 날씨 정보를 세션 상태에 저장
        st.session_state['weather'] = f"현재 [{location}] 날씨는 [{weather_desc}] 입니다."

    except requests.exceptions.RequestException as e:
        # 네트워크 오류 발생 시
        st.session_state['weather'] = f"오류: 날씨 정보를 가져오는 데 실패했습니다. (네트워크 문제) - {e}"
    except AttributeError:
        # HTML 구조가 변경되어 정보를 찾지 못할 경우
        st.session_state['weather'] = "오류: 웹사이트 구조가 변경되어 날씨 정보를 찾을 수 없습니다."
    except Exception as e:
        # 그 외 모든 예외 처리
        st.session_state['weather'] = f"알 수 없는 오류가 발생했습니다: {e}"


# --- UI (사용자 인터페이스) ---
st.title("🌤️ 오늘의 날씨 알리미")
st.write("버튼을 눌러 현재 서울의 날씨를 확인