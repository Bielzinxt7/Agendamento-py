# Agendamento-py
Sistema de agendamento médico.
import tkinter as tk
from tkinter import ttk, messagebox
# Certifique-se de que a linha abaixo não está comentada!
from tkcalendar import Calendar # Você precisa instalar: pip install tkcalendar 
import random
from datetime import datetime

# --- Dados Simulados (simulando um "banco de dados") ---
CIDADES = ["São Paulo", "Rio de Janeiro", "Belo Horizonte", "Porto Alegre", "Curitiba"]
MEDICOS_DISPONIVEIS = {
    "São Paulo": ["Dr. Silva (Clínico Geral)", "Dra. Souza (Pediatra)"],
    "Rio de Janeiro": ["Dr. Oliveira (Cardiologista)", "Dra. Costa (Dermatologista)"],
    "Belo Horizonte": ["Dr. Santos (Ortopedista)", "Dra. Lima (Ginecologista)"],
    "Porto Alegre": ["Dra. Ferreira (Ginecologista)"],
    "Curitiba": ["Dr. Almeida (Neurologista)", "Dra. Rocha (Endocrinologista)"]
}
HORARIOS_DISPONIVEIS = ["08:00", "09:00", "10:00", "11:00", "14:00", "15:00", "16:00"]
TOTAL_ATENDIMENTOS = 1005 # Contador de atendimentos global simulado

# Variáveis globais para armazenar os dados do agendamento
dados_agendamento = {
    "cidade": "",
    "data_selecionada": "",
    "horario": "",
    "medico": ""
}
dados_pessoais = {
    "nome": "",
    "cpf": "",
    "telefone": "",
    "email": ""
}

class AplicativoAgendamento(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Sistema de Agendamento Médico")
        self.geometry("600x450")
        
        self.container = ttk.Frame(self)
        self.container.pack(fill="both", expand=True)
        self.frames = {}
        
        self.criar_telas()
        self.mostrar_tela("TelaAgendamento")

    def criar_telas(self):
        tela_agendamento = TelaAgendamento(self.container, self)
        self.frames["TelaAgendamento"] = tela_agendamento
        tela_agendamento.grid(row=0, column=0, sticky="nsew")

        tela_dados = TelaDadosPessoais(self.container, self)
        self.frames["TelaDadosPessoais"] = tela_dados
        tela_dados.grid(row=0, column=0, sticky="nsew")

        tela_sucesso = TelaSucesso(self.container, self)
        self.frames["TelaSucesso"] = tela_sucesso
        tela_sucesso.grid(row=0, column=0, sticky="nsew")

        self.container.grid_rowconfigure(0, weight=1)
        self.container.grid_columnconfigure(0, weight=1)

    def mostrar_tela(self, nome_tela):
        frame = self.frames[nome_tela]
        frame.tkraise()

    def finalizar_agendamento(self):
        global TOTAL_ATENDIMENTOS
        TOTAL_ATENDIMENTOS += 1
        
        self.frames["TelaSucesso"].mostrar_detalhes()
        self.mostrar_tela("TelaSucesso")

# --- Tela 1: Seleção de Agendamento ---
class TelaAgendamento(ttk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent)
        self.controller = controller
        self.data_selecionada = None

        ttk.Label(self, text="Agendamento de Consulta", font=("Arial", 16, "bold")).pack(pady=20)

        # 1. Selecionar Cidade
        ttk.Label(self, text="Selecione a Cidade:").pack(pady=5)
        self.var_cidade = tk.StringVar(self)
        self.combo_cidade = ttk.Combobox(self, textvariable=self.var_cidade, values=CIDADES, state="readonly")
        self.combo_cidade.bind("<<ComboboxSelected>>", self.atualizar_medicos)
        self.combo_cidade.pack(pady=5)
        self.var_cidade.set(CIDADES[0] if CIDADES else "")

        # 2. Selecionar Data
        ttk.Label(self, text="Selecione a Data:").pack(pady=5)
        self.btn_data = ttk.Button(self, text="Abrir Calendário", command=self.abrir_calendario)
        self.btn_data.pack(pady=5)
        self.lbl_data_selecionada = ttk.Label(self, text="Nenhuma data selecionada")
        self.lbl_data_selecionada.pack(pady=5)
        

        # 3. Selecionar Médico Disponível
        ttk.Label(self, text="Médico Disponível:").pack(pady=5)
        self.var_medico = tk.StringVar(self)
        self.combo_medico = ttk.Combobox(self, textvariable=self.var_medico, state="readonly")
        self.combo_medico.pack(pady=5)
        self.atualizar_medicos()

        # 4. Selecionar Horário
        ttk.Label(self, text="Selecione o Horário:").pack(pady=5)
        self.var_horario = tk.StringVar(self)
        self.combo_horario = ttk.Combobox(self, textvariable=self.var_horario, values=HORARIOS_DISPONIVEIS, state="readonly")
        self.combo_horario.pack(pady=5)

        # Botão Próxima Tela
        ttk.Button(self, text="Próximo (Dados Pessoais)", command=self.avancar_para_dados).pack(pady=20)

    def atualizar_medicos(self, event=None):
        cidade = self.var_cidade.get()
        medicos = MEDICOS_DISPONIVEIS.get(cidade, [])
        self.combo_medico['values'] = medicos
        self.var_medico.set(medicos[0] if medicos else "")
        if not medicos:
             self.var_medico.set("Nenhum médico disponível")


    def abrir_calendario(self):
        """Abre uma nova janela para selecionar a data."""
        top = tk.Toplevel(self)
        top.title("Selecione a Data")
        
        # *** LINHA CORRIGIDA: SEM O ARGUMENTO 'locale' ***
        try:
            cal = Calendar(top, selectmode='day', date_pattern='dd/mm/yyyy')
        except Exception as e:
            messagebox.showerror("Erro no Calendário", f"Erro ao criar o calendário. Por favor, verifique se instalou 'tkcalendar' com 'pip install tkcalendar'. Detalhe: {e}")
            top.destroy()
            return

        cal.pack(pady=20, padx=20)

        def selecionar_data():
            self.data_selecionada = cal.get_date()
            self.lbl_data_selecionada.config(text=f"Data Selecionada: {self.data_selecionada}")
            top.destroy()
        
        ttk.Button(top, text="Confirmar Data", command=selecionar_data).pack(pady=10)

    def avancar_para_dados(self):
        cidade = self.var_cidade.get()
        data = self.data_selecionada
        horario = self.var_horario.get()
        medico = self.var_medico.get()

        if not (cidade and data and horario and medico):
            messagebox.showwarning("Erro de Validação", "Por favor, preencha todos os campos de agendamento: Cidade, Data, Horário e Médico.")
            return

        # Armazena os dados
        dados_agendamento["cidade"] = cidade
        dados_agendamento["data_selecionada"] = data
        dados_agendamento["horario"] = horario
        dados_agendamento["medico"] = medico
        
        self.controller.mostrar_tela("TelaDadosPessoais")

# --- Tela 2: Cadastro de Dados Pessoais ---
class TelaDadosPessoais(ttk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent)
        self.controller = controller

        ttk.Label(self, text="Dados Pessoais do Paciente", font=("Arial", 16, "bold")).pack(pady=20)

        campos = ["Nome:", "CPF:", "Telefone:", "Email:"]
        self.entries = {}

        for campo in campos:
            frame = ttk.Frame(self)
            frame.pack(pady=5, padx=10, fill='x')
            
            ttk.Label(frame, text=campo, width=15, anchor="w").pack(side="left", padx=5)
            entry = ttk.Entry(frame, width=40)
            entry.pack(side="right", expand=True, fill='x', padx=5)
            self.entries[campo.strip(':').lower()] = entry

        frame_botoes = ttk.Frame(self)
        frame_botoes.pack(pady=20)
        
        ttk.Button(frame_botoes, text="Voltar", command=lambda: controller.mostrar_tela("TelaAgendamento")).pack(side="left", padx=10)
        ttk.Button(frame_botoes, text="Confirmar Agendamento", command=self.confirmar_e_finalizar).pack(side="left", padx=10)

    def validar_e_armazenar(self, nome, cpf, telefone, email):
        if not (nome and cpf and telefone and email):
            messagebox.showwarning("Erro de Validação", "Por favor, preencha todos os campos de dados pessoais.")
            return False
        
        if "@" not in email or "." not in email:
            messagebox.showwarning("Erro de Validação", "O formato do e-mail é inválido.")
            return False

        dados_pessoais["nome"] = nome
        dados_pessoais["cpf"] = cpf
        dados_pessoais["telefone"] = telefone
        dados_pessoais["email"] = email
        return True

    def confirmar_e_finalizar(self):
        nome = self.entries["nome"].get()
        cpf = self.entries["cpf"].get()
        telefone = self.entries["telefone"].get()
        email = self.entries["email"].get()
        
        if self.validar_e_armazenar(nome, cpf, telefone, email):
            self.controller.finalizar_agendamento()

# --- Tela 3: Sucesso ---
class TelaSucesso(ttk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent)
        self.controller = controller

        ttk.Label(self, text="Agendamento Concluído!", font=("Arial", 18, "bold"), foreground="green").pack(pady=30)
        ttk.Label(self, text="Sua consulta foi agendada com sucesso!", font=("Arial", 12)).pack(pady=5)
        
        self.lbl_detalhes = ttk.Label(self, text="", font=("Arial", 11), justify=tk.LEFT)
        self.lbl_detalhes.pack(pady=15, padx=50, anchor="w")

        self.lbl_contador = ttk.Label(self, text="", font=("Arial", 12, "italic"), foreground="blue")
        self.lbl_contador.pack(pady=10)
        
        ttk.Button(self, text="Fazer Novo Agendamento", command=self.reiniciar_aplicativo).pack(pady=30)

    def mostrar_detalhes(self):
        detalhes = f"""
        **Dados do Paciente:**
        Nome: {dados_pessoais.get('nome')}
        CPF: {dados_pessoais.get('cpf')}
        Telefone: {dados_pessoais.get('telefone')}
        E-mail: {dados_pessoais.get('email')}
        
        **Detalhes da Consulta:**
        Cidade: {dados_agendamento.get('cidade')}
        Data: {dados_agendamento.get('data_selecionada')}
        Horário: {dados_agendamento.get('horario')}
        Médico: {dados_agendamento.get('medico')}
        """
        self.lbl_detalhes.config(text=detalhes.strip())
        self.lbl_contador.config(text=f"Você é o agendamento número: {TOTAL_ATENDIMENTOS}")

    def reiniciar_aplicativo(self):
        self.controller.mostrar_tela("TelaAgendamento")

# --- Inicialização do Aplicativo ---
if __name__ == "__main__":
    app = AplicativoAgendamento()
    app.mainloop()
