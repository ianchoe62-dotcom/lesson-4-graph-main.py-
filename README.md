import pandas as pd
import plotly.express as px
import streamlit as st

st.set_page_config(page_title="영화 데이터 그래프 도감 2 - 분포와 관계", layout="wide")
st.title("영화 데이터 그래프 도감 2 - 분포와 관계")

# 데이터 URL 변경
DATA_URL = (
    "https://raw.githubusercontent.com/happykth/data/main/kobis_daily.csv"
)


@st.cache_data
def load_data():
    # 일별 박스오피스 데이터를 불러옵니다.
    df = pd.read_csv(DATA_URL)

    # 장르 컬럼 처리 (genre 컬럼이 존재할 경우 세로막대 기호(|)의 첫 번째 장르 선택)
    if "genre" in df.columns:
        df["장르"] = df["genre"].fillna("기타").str.split("|").str[0]
    elif "genreAlt" in df.columns:
        df["장르"] = df["genreAlt"].fillna("기타").str.split("|").str[0]
    else:
        df["장르"] = "미지정"

    # 수치형 데이터 변환 (문자열 타입인 경우 처리)
    for col in ["audiCnt", "audiAcc", "scrnCnt", "showCnt"]:
        if col in df.columns:
            df[col] = pd.to_numeric(df[col], errors="coerce").fillna(0)

    return df


df = load_data()

# 주요 컬럼 자동 매핑
title_col = (
    "movieNm"
    if "movieNm" in df.columns
    else ("영화명" if "영화명" in df.columns else df.columns[0])
)
audi_col = (
    "audiAcc"
    if "audiAcc" in df.columns
    else ("audiCnt" if "audiCnt" in df.columns else None)
)
scrn_col = (
    "scrnCnt"
    if "scrnCnt" in df.columns
    else ("스크린수" if "스크린수" in df.columns else None)
)


# ── 그래프 1. 장르별 영화 편수 (도넛) ──
st.header("1. 장르별 영화 편수 (도넛)")

# 영화별 중복을 제거하여 실제 영화 단위 편수를 계산합니다.
movie_df = df.drop_duplicates(subset=[title_col]) if title_col else df
genre_count = movie_df["장르"].value_counts().reset_index()
genre_count.columns = ["장르", "편수"]

fig1 = px.pie(
    genre_count,
    names="장르",
    values="편수",
    hole=0.45,  # 가운데 구멍을 뚫어 도넛 모양으로 설정
)
fig1.update_traces(hovertemplate="%{label}<br>%{value}편 (%{percent})<extra></extra>")
st.plotly_chart(fig1, use_container_width=True)

st.text_input("이 그래프로 알 수 있는 것", key="note1")

st.divider()


# ── 그래프 2. 장르별 관객수 분포 (상자 그림) ──
st.header("2. 장르별 관객수 분포 (상자 그림)")
st.caption(
    "장르별 영화들의 누적 관객수 중앙값, 사분위수 및 대흥행작(아웃라이어)을 확인합니다."
)

if audi_col:
    # 영화별 최대 누적 관객수 집계
    movie_audi = (
        movie_df.groupby(["장르", title_col])[audi_col].max().reset_index()
    )

    fig2 = px.box(
        movie_audi,
        x="장르",
        y=audi_col,
        color="장르",
        points="all",  # 개별 영화 데이터 포인트를 함께 표시
        hover_name=title_col,
        labels={audi_col: "누적 관객수(명)", "장르": "영화 장르"},
    )
    fig2.update_layout(showlegend=False)
    st.plotly_chart(fig2, use_container_width=True)

st.text_input("이 그래프로 알 수 있는 것", key="note2")

st.divider()


# ── 그래프 3. 스크린수와 관객수의 관계 (산점도) ──
st.header("3. 스크린수와 관객수의 관계 (산점도)")
st.caption("스크린 확보 수와 누적 관객수 간의 상관관계를 관찰합니다.")

if scrn_col and audi_col:
    # 영화별 최대 스크린수 및 최대 누적 관객수 집계
    movie_relation = (
        movie_df.groupby(["장르", title_col])[[scrn_col, audi_col]]
        .max()
        .reset_index()
    )

    fig3 = px.scatter(
        movie_relation,
        x=scrn_col,
        y=audi_col,
        color="장르",
        hover_name=title_col,
        labels={scrn_col: "최대 스크린수(개)", audi_col: "누적 관객수(명)"},
        trendline="ols",  # 전체 경향 추세선 표시
    )
    st.plotly_chart(fig3, use_container_width=True)

st.text_input("이 그래프로 알 수 있는 것", key="note3")# lesson-4-graph-main.py-
