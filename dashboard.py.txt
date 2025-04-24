import streamlit as st
import pandas as pd

# Título geral
st.set_page_config(page_title="Dashboard Coordenati Tatuí", layout="wide")
st.title("📊 Dashboard de Anúncios - Coordenati Tatuí")

# Função para carregar e filtrar dados
def carregar_dados(caminho, tipo_metrica):
    df = pd.read_csv(caminho)
    df["Data de início"] = pd.to_datetime(df["Data de início"], errors="coerce")
    df["Data de término"] = pd.to_datetime(df["Data de término"], errors="coerce")
    
    if tipo_metrica == "thruplay":
        df = df[df["Indicador de resultados"] == "Visualizações de vídeo com reprodução de 15 segundos"]
    elif tipo_metrica == "conversas":
        df = df[df["Indicador de resultados"] == "Conversas iniciadas"]
    
    df["CTR (link)"] = df["Cliques no link"] / df["Impressões"]
    
    return df

# Abas: captação e conversão
aba = st.tabs(["📣 Campanha de Captação", "💬 Campanha de Conversão"])

# ------------------------ CAPTAÇÃO ------------------------
with aba[0]:
    st.header("📣 Captação - Engajamento e Thruplay")

    # Upload ou leitura local
    df_capt1 = carregar_dados("CA---Coordenati-Tatuí-Anúncios-20-de-mar-de-2025-7-de-a-r-de-2025.csv", "thruplay")
    df_capt2 = carregar_dados("CA---Coordenati-Tatuí-Anúncios-8-de-a-r-de-2025-20-de-a-r-de-2025 - Captação.csv", "thruplay")
    df_cap = pd.concat([df_capt1, df_capt2], ignore_index=True)

    st.subheader("🎯 Filtros")
    with st.expander("Filtrar dados"):
        campanhas = st.multiselect("Campanhas", options=df_cap["Nome da campanha"].unique(), default=None)
        ativos = st.multiselect("Status de veiculação", options=df_cap["Veiculação"].unique(), default=None)
        data_inicio, data_fim = st.date_input("Período", [df_cap["Data de início"].min(), df_cap["Data de término"].max()])

    if campanhas:
        df_cap = df_cap[df_cap["Nome da campanha"].isin(campanhas)]
    if ativos:
        df_cap = df_cap[df_cap["Veiculação"].isin(ativos)]
    df_cap = df_cap[(df_cap["Data de início"] >= pd.to_datetime(data_inicio)) & (df_cap["Data de término"] <= pd.to_datetime(data_fim))]

    st.dataframe(df_cap[[
        "Nome do anúncio", "Veiculação", "Impressões", "Alcance", 
        "Cliques (todos)", "Cliques no link", "CTR (link)", 
        "Resultados", "Custo por resultados", "Valor gasto (BRL)"
    ]].sort_values("Resultados", ascending=False), use_container_width=True)

# ------------------------ CONVERSÃO ------------------------
with aba[1]:
    st.header("💬 Conversão - Mensagens no WhatsApp")

    df_conj = carregar_dados("CA---Coordenati-Tatuí-Conjuntos-de-anúncios-8-de-a-r-de-2025-20-de-a-r-de-2025 Conj Conversão.csv", "conversas")
    df_cria = carregar_dados("CA---Coordenati-Tatuí-Anúncios-8-de-a-r-de-2025-20-de-a-r-de-2025 Criat Conversão.csv", "conversas")
    df_conv = pd.concat([df_conj, df_cria], ignore_index=True)

    st.subheader("🎯 Filtros")
    with st.expander("Filtrar dados"):
        campanhas = st.multiselect("Campanhas", options=df_conv["Nome da campanha"].unique(), default=None)
        ativos = st.multiselect("Status de veiculação", options=df_conv["Veiculação"].unique(), default=None)
        data_inicio, data_fim = st.date_input("Período", [df_conv["Data de início"].min(), df_conv["Data de término"].max()])

    if campanhas:
        df_conv = df_conv[df_conv["Nome da campanha"].isin(campanhas)]
    if ativos:
        df_conv = df_conv[df_conv["Veiculação"].isin(ativos)]
    df_conv = df_conv[(df_conv["Data de início"] >= pd.to_datetime(data_inicio)) & (df_conv["Data de término"] <= pd.to_datetime(data_fim))]

    st.dataframe(df_conv[[
        "Nome do anúncio", "Veiculação", "Impressões", "Alcance", 
        "Cliques (todos)", "Cliques no link", "CTR (link)", 
        "Resultados", "Custo por resultados", "Valor gasto (BRL)"
    ]].sort_values("Resultados", ascending=False), use_container_width=True)
