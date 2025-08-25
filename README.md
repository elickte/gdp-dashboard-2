# streamlit 라이브러리를 불러옵니다.
import streamlit as st

# --- 세션 상태 초기화 ---
# 앱이 처음 실행되거나 페이지가 새로고침될 때, st.session_state에 필요한 값들이 없으면 초기화합니다.
# 이렇게 하면 사용자의 선택이나 앱의 상태를 페이지 이동이나 상호작용 간에 유지할 수 있습니다.
if 'page' not in st.session_state:
    st.session_state.page = 'home'  # 현재 페이지 상태 (home: 시작, result: 결과)
if 'answers' not in st.session_state:
    st.session_state.answers = {}  # 사용자의 답변을 저장할 딕셔너리
if 'mbti_type' not in st.session_state:
    st.session_state.mbti_type = '' # 최종 MBTI 유형을 저장할 변수

# --- 데이터 정의 ---
# MBTI 각 지표에 대한 질문과 선택지를 정의합니다.
questions = {
    "EI": {
        "question": "휴식을 취할 때 주로 어떤 활동을 선호하시나요?",
        "options": {"E": "친구들과 만나거나 활발한 활동하기", "I": "혼자 조용히 책을 읽거나 영화보기"},
    },
    "SN": {
        "question": "새로운 정보를 접할 때 어떤 방식에 더 끌리시나요?",
        "options": {"S": "실제 경험과 구체적인 사실에 집중하기", "N": "미래의 가능성과 숨겨진 의미를 상상하기"},
    },
    "TF": {
        "question": "결정을 내릴 때 무엇을 더 중요하게 생각하시나요?",
        "options": {"T": "논리적인 분석과 객관적인 사실", "F": "주변 사람들과의 관계와 개인적인 감정"},
    },
    "JP": {
        "question": "여행을 계획할 때 어떤 스타일이신가요?",
        "options": {"J": "출발 전 꼼꼼하게 일정을 짜고 계획대로 움직이기", "P": "정해진 계획 없이 상황에 따라 즉흥적으로 즐기기"},
    }
}

# 16가지 MBTI 유형별 학습 스타일 설명 및 추천 도서 장르를 정의합니다.
mbti_info = {
    "ISTJ": {"style": "체계적이고 사실에 기반한 학습을 선호합니다. 조용하고 질서 있는 환경에서 혼자 공부할 때 집중력이 높습니다.", "books": "역사, 논픽션, 실용서"},
    "ISFJ": {"style": "구체적인 정보를 바탕으로 꾸준하고 성실하게 학습합니다. 다른 사람에게 도움을 주는 과정에서 학습 효과가 오릅니다.", "books": "가족 소설, 요리책, 전기"},
    "INFJ": {"style": "통찰력을 바탕으로 의미를 찾고, 아이디어를 깊이 탐구하는 학습을 즐깁니다. 인류에 기여할 수 있는 주제에 관심이 많습니다.", "books": "심리학, 철학, 시집"},
    "INTJ": {"style": "논리적이고 복잡한 이론을 탐구하는 것을 좋아합니다. 독립적으로 학습하며 비전을 세우고 지식을 체계화하는 데 능숙합니다.", "books": "과학, 전략, SF 소설"},
    "ISTP": {"style": "직접 손으로 만지고 경험하는 실습 위주의 학습에서 뛰어난 능력을 보입니다. 문제 해결 과정 자체를 즐깁니다.", "books": "DIY, 과학 스릴러, 탐정 소설"},
    "ISFP": {"style": "자유로운 분위기에서 자신의 가치와 조화를 이루는 주제를 학습할 때 흥미를 느낍니다. 미적 감각이 뛰어납니다.", "books": "예술, 여행 에세이, 판타지 소설"},
    "INFP": {"style": "자신의 이상과 가치에 부합하는 내용을 학습할 때 동기부여가 됩니다. 창의적이고 상상력이 풍부합니다.", "books": "판타지, 시집, 성장 소설"},
    "INTP": {"style": "논리적 분석과 추상적인 개념을 다루는 것을 좋아합니다. 지적 호기심이 강하며, 독립적인 학습 환경을 선호합니다.", "books": "SF 소설, 철학, 미스터리"},
    "ESTP": {"style": "활동적이고 실제적인 경험을 통해 배우는 것을 선호합니다. 이론보다는 실전에 강하며, 친구들과 함께 학습하는 것을 즐깁니다.", "books": "모험, 액션, 창업 관련 서적"},
    "ESFP": {"style": "즐겁고 사교적인 분위기에서 사람들과 상호작용하며 학습할 때 효과적입니다. 실생활에 바로 적용할 수 있는 지식을 좋아합니다.", "books": "로맨틱 코미디, 연예, 패션 잡지"},
    "ENFP": {"style": "열정적이고 창의적인 아이디어를 탐구하는 것을 즐깁니다. 다양한 가능성을 상상하며, 토론과 같은 상호작용이 활발한 학습을 선호합니다.", "books": "인문학, 자기계발, 여행기"},
    "ENTP": {"style": "지적인 도전을 즐기며, 기존의 방식에 의문을 제기하고 논쟁하는 것을 좋아합니다. 복잡한 문제를 해결하는 데 능합니다.", "books": "토론서, 과학 논픽션, 비평"},
    "ESTJ": {"style": "체계적이고 조직적인 환경에서 명확한 목표를 가지고 학습하는 것을 선호합니다. 실용적이고 논리적인 정보를 잘 다룹니다.", "books": "경영, 경제, 실용 가이드"},
    "ESFJ": {"style": "다른 사람들과 협력하고 조화로운 분위기에서 학습할 때 능률이 오릅니다. 타인의 인정과 칭찬이 중요한 동기부여가 됩니다.", "books": "인간관계, 소설, 요리책"},
    "ENFJ": {"style": "사람들과의 상호작용 속에서 영감을 얻으며, 다른 사람의 성장을 돕는 학습을 선호합니다. 비전과 열정이 넘칩니다.", "books": "리더십, 심리학, 교육"},
    "ENTJ": {"style": "전략적이고 목표 지향적인 학습을 선호합니다. 효율성을 중요시하며, 지식을 활용하여 미래를 계획하고 통솔하는 데 능숙합니다.", "books": "전략, 리더십, 정치"},
}

# --- 페이지 렌더링 함수 ---
# 시작 페이지를 표시하는 함수
def render_home():
    st.title("🚀 MBTI 기반 학습 유형 진단")
    st.write("간단한 네 가지 질문에 답변하고 자신에게 맞는 학습 스타일을 알아보세요!")
    
    # questions 딕셔너리를 순회하며 각 질문과 선택지를 화면에 표시합니다.
    for key, value in questions.items():
        # st.radio를 사용해 사용자에게 질문과 선택지를 보여주고, 선택값을 st.session_state.answers에 저장합니다.
        st.session_state.answers[key] = st.radio(
            value["question"],
            options=list(value["options"].values()),
            key=f"q_{key}"
        )
    
    # "진단 결과 보기" 버튼을 만듭니다. 이 버튼을 누르면 st.session_state.page 값이 'result'로 변경됩니다.
    if st.button("진단 결과 보기"):
        st.session_state.page = "result"
        # 페이지가 즉시 새로고침되어 결과 페이지로 이동하도록 합니다.
        st.rerun()

# 결과 페이지를 표시하는 함수
def render_result():
    # 사용자의 답변을 조합하여 MBTI 유형을 계산합니다.
    mbti = ""
    for key, value in questions.items():
        # 사용자가 선택한 답변(value)이 어떤 키(E, I, S, N...)에 해당하는지 찾습니다.
        for mbti_char, option_text in value["options"].items():
            if st.session_state.answers[key] == option_text:
                mbti += mbti_char
                break
    st.session_state.mbti_type = mbti

    # 결과 페이지의 제목을 표시합니다.
    st.title(f"🔍 당신의 학습 유형은 '{st.session_state.mbti_type}' 입니다!")

    # 계산된 MBTI 유형에 맞는 정보를 mbti_info 딕셔너리에서 가져와 화면에 표시합니다.
    if st.session_state.mbti_type in mbti_info:
        info = mbti_info[st.session_state.mbti_type]
        st.subheader("💡 학습 스타일")
        st.write(info["style"])
        
        st.subheader("📚 추천 도서 장르")
        st.write(info["books"])
        
        st.markdown("---")
        st.info("추천 장르의 책들을 교보문고에서 찾아보세요!")
        # 교보문고 링크를 제공합니다. 사용자가 클릭하면 새 탭에서 열립니다.
        st.link_button("교보문고에서 책 찾아보기", "https://www.kyobobook.co.kr/?utm_source=google&utm_medium=cpc&utm_campaign=googleSearch&gt_network=g&gt_keyword=%EA%B5%90%EB%B3%B4%EB%AC%B8%EA%B3%A0&gt_target_id=kwd-320569486700&gt_campaign_id=9979905549&gt_adgroup_id=98010719662&gad_source=1")

    # "다시 진단하기" 버튼을 만듭니다.
    if st.button("다시 진단하기"):
        # 세션 상태를 초기화하여 처음부터 다시 시작할 수 있도록 합니다.
        st.session_state.page = "home"
        st.session_state.answers = {}
        st.session_state.mbti_type = ""
        # 페이지를 새로고침합니다.
        st.rerun()

# --- 메인 로직 ---
# st.session_state.page 값에 따라 적절한 페이지 렌더링 함수를 호출합니다.
if st.session_state.page == 'home':
    render_home()
elif st.session_state.page == 'result':
    render_result()