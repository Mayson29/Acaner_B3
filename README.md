# Acaner_B3
# app.py
import streamlit as st
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
from io import BytesIO

st.set_page_config(page_title="Scanner B3 Exponencial", layout="wide")
st.title("Scanner de Investimentos Exponenciais - B3")

# Parâmetros do usuário
st.sidebar.header("Filtros de Crescimento")
min_crescimento = st.sidebar.slider("Crescimento mínimo (%)", 0, 100, 15)
anos = st.sidebar.slider("Anos de histórico", 3, 5, 4)

# Empresas da B3 (adicione mais se quiser)
empresas = {
    "PETR4": "Petrobras",
    "VALE3": "Vale",
    "ITUB4": "Itaú Unibanco",
    "WEGE3": "Weg",
    "MGLU3": "Magazine Luiza",
    "B3SA3": "B3",
    "RENT3": "Localiza",
    "LREN3": "Lojas Renner"
}

def crescimento_exponencial(valores):
    taxas = []
    for i in range(1, len(valores)):
        if valores[i-1] == 0:
            return 0, False
        taxa = (valores[i] / valores[i-1]) - 1
        taxas.append(taxa)
    media = sum(taxas) / len(taxas)
    return media, all(t > 0 for t in taxas)

def gerar_grafico(receita, lucro, nome):
    fig, ax = plt.subplots()
    ax.plot(receita, label="Receita", marker='o')
    ax.plot(lucro, label="Lucro Líquido", marker='o')
    ax.set_title(f"{nome} - Receita vs Lucro")
    ax.legend()
    st.pyplot(fig)

def empresa_promissora(ticker):
    try:
        acao = yf.Ticker(ticker + ".SA")
        income = acao.income_stmt
        lucro = income.loc['Net Income'].dropna().values[::-1]
        receita = income.loc['Total Revenue'].dropna().values[::-1]

        if len(lucro) < anos or len(receita) < anos:
            return None

        lucro = lucro[-anos:]
        receita = receita[-anos:]

        cresc_r, pos_r = crescimento_exponencial(receita)
        cresc_l, pos_l = crescimento_exponencial(lucro)

        if pos_r and pos_l and cresc_r > min_crescimento / 100 and cresc_l > min_crescimento / 100:
            return {
                "Ticker": ticker,
                "Nome": empresas[ticker],
                "Crescimento Receita (%)": round(cresc_r * 100, 2),
                "Crescimento Lucro (%)": round(cresc_l * 100, 2),
                "Receita": receita.tolist(),
                "Lucro": lucro.tolist()
            }
    except:
        return None
    return None

st.markdown("Clique no botão abaixo para rodar a análise com os filtros selecionados:")

if st.button("Analisar Empresas da B3"):
    resultados = []
    for ticker in empresas.keys():
        resultado = empresa_promissora(ticker)
        if resultado:
            resultados.append(resultado)

    if resultados:
        df = pd.DataFrame([{
            "Ticker": r["Ticker"],
            "Nome": r["Nome"],
            "Crescimento Receita (%)": r["Crescimento Receita (%)"],
            "Crescimento Lucro (%)": r["Crescimento Lucro (%)"]
        } for r in resultados])

        st.success(f"{len(df)} empresas encontradas.")
        st.dataframe(df)

        for r in resultados:
            st.subheader(r["Nome"])
            gerar_grafico(r["Receita"], r["Lucro"], r["Nome"])

        # Download Excel
        output = BytesIO()
        df.to_excel(output, index=False)
        output.seek(0)
        st.download_button(
            label="Baixar resultados em Excel",
            data=output,
            file_name="empresas_exponenciais_b3.xlsx",
            mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
        )

    else:
        st.warning("Nenhuma empresa com crescimento exponencial dentro dos critérios."
streamlit
yfinance
pandas
matplotlib
openpyxl
