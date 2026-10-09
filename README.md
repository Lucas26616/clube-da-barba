import streamlit as st
import datetime

# Configuração da página e do tema visual (Preto Grafite e Dourado)
st.set_page_config(
    page_title="Clube da Barbearia - Agendamentos", 
    page_icon="💈", 
    layout="centered"
)

# Injeção de CSS nativo para forçar as cores personalizadas sem quebrar o layout
st.markdown("""
 <style>
 /* Fundo da página (Preto Grafite) */
 .stApp {
     background-color: #1A1A1A !important;
     color: #FFFFFF !important;
 }
 
 /* Títulos e Subtítulos em Dourado */
 h1, h2, h3, h4, h5, h6, .stSubheader, p {
     color: #D4AF37 !important;
 ]
 
 /* Customização dos botões (Fundo dourado, texto escuro) */
 div.stButton > button {
     background-color: #D4AF37 !important;
     color: #1A1A1A !important;
     font-weight: bold !important;
     border: none !important;
     width: 100% !important;
 }
 </style>
""", unsafe_allow_html=True)

# Inicialização do "Banco de Dados" simulado na sessão do usuário
if "agendamentos" not in st.session_state:
    st.session_state.agendamentos = []

# Informações da Barbearia
SERVICOS = {
    "Corte de Cabelo": 30,
    "Barba": 30,
    "Sobrancelha": 30,
    "Prótese Capilar": 60
}

HORA_INICIO = 8
HORA_FIM = 18

def gerar_horarios_dia():
    horarios = []
    inicio = datetime.datetime.combine(datetime.date.today(), datetime.time(HORA_INICIO, 0))
    limite = datetime.datetime.combine(datetime.date.today(), datetime.time(HORA_FIM, 0))
    
    while inicio < limite:
        horarios.append(inicio.time().strftime("%H:%M"))
        inicio += datetime.timedelta(minutes=30)
    return horarios

# Interface do Usuário
st.title("💈 Clube da Barbearia")
st.subheader("Reserve o seu horário de forma simples e rápida")

# Abas para organizar a visualização
aba_reservar, aba_meus_agendamentos = st.tabs(["📅 Reservar Horário", "📋 Agendamentos do Dia"])

with aba_reservar:
    # 1. Seleção do Serviço
    servico_selecionado = st.selectbox(
        "Selecione o serviço desejado:",
        options=list(SERVICOS.keys()),
        format_func=lambda x: f"{x} - R$ {SERVICOS[x]},00"
    )
    
    # 2. Dados do Cliente
    nome_cliente = st.text_input("Seu Nome:")
    telefone_cliente = st.text_input("Seu Telefone / WhatsApp:")
    
    # 3. Seleção de Data e Horário
    col1, col2 = st.columns(2)
    
    with col1:
        data_selecionada = st.date_input("Escolha a data:", min_value=datetime.date.today())
        
    with col2:
        horarios_disponiveis = gerar_horarios_dia()
        horarios_ocupados = [
            a["hora"] for a in st.session_state.agendamentos 
            if a["data"] == data_selecionada.strftime("%d/%m/%Y")
        ]
        horarios_filtrados = [h for h in horarios_disponiveis if h not in horarios_ocupados]
        
        if horarios_filtrados:
            hora_selecionada = st.selectbox("Escolha o horário:", options=horarios_filtrados)
        else:
            st.warning("Não há horários disponíveis para este dia.")
            hora_selecionada = None

    # 4. Botão de Confirmação
    if st.button("Confirmar Agendamento"):
        if not nome_cliente.strip():
            st.error("Por favor, insira o seu nome.")
        elif not telefone_cliente.strip():
            st.error("Por favor, insira o seu telefone.")
        elif hora_selecionada is None:
            st.error("Selecione um horário válido.")
        else:
            novo_agendamento = {
                "cliente": nome_cliente,
                "telefone": telefone_cliente,
                "servico": servico_selecionado,
                "data": data_selecionada.strftime("%d/%m/%Y"),
                "hora": hora_selecionada
            }
            st.session_state.agendamentos.append(novo_agendamento)
            st.success(f"Agendamento realizado para {novo_agendamento['data']} às {novo_agendamento['hora']}!")

with aba_meus_agendamentos:
    if not st.session_state.agendamentos:
        st.info("Nenhum agendamento realizado até o momento.")
    else:
        for agendamento in st.session_state.agendamentos:
            st.code(f"Cliente: {agendamento['cliente']} | Serviço: {agendamento['servico']} | Horário: {agendamento['data']} às {agendamento['hora']}")
