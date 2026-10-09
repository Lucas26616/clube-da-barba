import streamlit as st
import datetime

# Configuração da página e do tema visual (Preto Grafite e Dourado)
st.set_page_config(
    page_title="Clube da Barbearia - Agendamentos", 
    page_icon="💈", 
    layout="centered"
)

# Injeção de CSS para forçar as cores personalizadas
st.markdown("""
 <style>
 /* Fundo da página (Preto Grafite) */
 .stApp {
     background-color: #1A1A1A;
     color: #FFFFFF;
 }
 
 /* Títulos e Subtítulos em Dourado */
 h1, h2, h3, h4, h5, h6, .stSubheader {
     color: #D4AF37 !important;
 }
 
 /* Customização dos botões (Fundo dourado, texto escuro) */
 div.stButton > button:first-child {
     background-color: #D4AF37;
     color: #1A1A1A;
     font-weight: bold;
     border: none;
     transition: all 0.3s ease;
     width: 100%;
 }
 div.stButton > button:first-child:hover {
     background-color: #AA8726;
     color: #FFFFFF;
     box-shadow: 0 4px 15px rgba(212, 175, 55, 0.4);
 }
 
 /* Inputs, Selectboxes e Datepickers text color */
 label, .stSelectbox, .stTextInput, .stDateInput {
     color: #FFFFFF !important;
 }
 
 /* Caixas de Alerta/Info personalizadas */
 .stAlert {
     background-color: #2D2D2D !important;
     border-left: 5px solid #D4AF37 !important;
     color: #FFFFFF !important;
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
    atual = datetime.datetime.combine(datetime.date.today(), datetime.time(HORA_INICIO, 0))
    limite = datetime.datetime.combine(datetime.date.today(), datetime.time(HORA_FIM, 0))
    
    while atual < limite:
        horarios.append(atual.time().strftime("%H:%M"))
        atual += datetime.timedelta(minutes=30)
    return horarios

# Interface do Usuário
st.title("💈 Clube da Barbearia")
st.subheader("Reserve o seu horário de forma simples e rápida")

# Abas para organizar a visualização
aba_reservar, aba_meus_agendamentos = st.tabs(["📅 Reservar Horário", "📋 Agendamentos do Dia"])

with aba_reservar:
    st.write("---")
    
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
        # Bloqueia datas passadas
        data_selecionada = st.date_input("Escolha a data:", min_value=datetime.date.today())
        
    with col2:
        # Filtra horários já ocupados naquele dia específico
        horarios_disponiveis = gerar_horarios_dia()
        horarios_ocupados = [
            a["hora"] for a in st.session_state.agendamentos 
            if a["data"] == data_selecionada.strftime("%d/%m/%Y")
        ]
        horarios_filtrados = [h for h in horarios_disponiveis if h not in horarios_ocupados]
        
        if horarios_filtrados:
            hora_selecionada = st.selectbox("Escolha o horário:", options=horarios_filtrados)
        else:
            st.warning("⚠️ Não há horários disponíveis para este dia.")
            hora_selecionada = None

    # 4. Botão de Confirmação
    if st.button("Confirmar Agendamento"):
        if not nome_cliente.strip():
            st.error("Por favor, insira o seu nome para realizar o agendamento.")
        elif not telefone_cliente.strip():
            st.error("Por favor, insira o seu telefone.")
        elif hora_selecionada is None:
            st.error("Não é possível agendar sem um horário selecionado.")
        else:
            # Salva o agendamento no session_state
            novo_agendamento = {
                "cliente": nome_cliente,
                "telefone": telefone_cliente,
                "servico": servico_selecionado,
                "data": data_selecionada.strftime("%d/%m/%Y"),
                "hora": hora_selecionada
            }
            st.session_state.agendamentos.append(novo_agendamento)
            st.success(f"🎉 Agendamento realizado com sucesso para {novo_agendamento['data']} às {novo_agendamento['hora']}!")
            st.balloons()

with aba_meus_agendamentos:
    st.write("---")
    if not st.session_state.agendamentos:
        st.info("Nenhum agendamento realizado até o momento.")
    else:
        # Exibe os agendamentos salvos organizados por cartões explicativos
        for i, agendamento in enumerate(st.session_state.agendamentos):
            st.markdown(f"""
            <div class='stAlert'>
                <strong>Cliente:</strong> {agendamento['cliente']}<br>
                <strong>Serviço:</strong> {agendamento['servico']}<br>
                <strong>Data:</strong> {agendamento['data']} às {agendamento['hora']}<br>
                <strong>Contato:</strong> {agendamento['telefone']}
            </div>
            <br>
            """, unsafe_allow_html=True)
