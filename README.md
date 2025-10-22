import tkinter as tk
from tkinter import ttk, messagebox
from tkcalendar import Calendar
# NOTA: Assumindo que essas variáveis e classes (AplicativoAgendamento, TelaAgendamento)
# estão definidas no seu código completo.
dados_agendamento = {}
dados_pessoais = {}
TOTAL_ATENDIMENTOS = 0 # Valor de exemplo, deve vir da sua lógica de controle.

class AplicativoAgendamento(tk.Tk):
    """Classe Principal da Aplicação - Controladora de Telas e Estilos."""
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.title("Sistema de Agendamento Médico")
        self.geometry("700x550") # Tamanho ajustado para o novo layout
        
        # 1. Configuração do Tema Moderno
        self.style = ttk.Style(self)
        # O tema 'clam' é um bom ponto de partida para customização moderna
        self.style.theme_use('clam') 
        
        # 2. Definição de Estilos Personalizados
        self.configurar_estilos()

        # Criação do Container de Telas
        container = ttk.Frame(self)
        container.pack(side="top", fill="both", expand=True)
        container.grid_rowconfigure(0, weight=1)
        container.grid_columnconfigure(0, weight=1)

        self.frames = {}
        
        # Criação e registro das Telas
        for F in (TelaAgendamento, TelaDadosPessoais, TelaSucesso):
            page_name = F.__name__
            # NOTA: TelaAgendamento precisa ser definida para rodar, estou assumindo que ela existe.
            if page_name == "TelaAgendamento":
                # Usando um placeholder, pois a definição da tela é extensa e está incompleta
                frame = ttk.Frame(container)
            else:
                frame = F(parent=container, controller=self)
            
            self.frames[page_name] = frame
            frame.grid(row=0, column=0, sticky="nsew")

        self.mostrar_tela("TelaAgendamento") # Inicia na primeira tela

    def configurar_estilos(self):
        # Estilo para Títulos (fonte maior, cor de destaque)
        self.style.configure('Titulo.TLabel', font=("Helvetica", 18, "bold"), foreground="#007bff") # Azul
        
        # Estilo para Subtítulos/Seções
        self.style.configure('Subtitulo.TLabel', font=("Helvetica", 14, "bold"), foreground="#333333")
        
        # Estilo para Labels de Campo (mais discreto)
        self.style.configure('Campo.TLabel', font=("Helvetica", 10), foreground="#555555")
        
        # Estilo para Botões Principais (Cor de destaque, padding maior)
        self.style.configure('Principal.TButton', font=("Helvetica", 11, "bold"), background="#28a745", foreground="white", padding=[15, 8]) # Verde
        self.style.map('Principal.TButton', 
                       background=[('active', '#218838'), ('disabled', '#6c757d')], # Escurece ao passar o mouse
                       foreground=[('active', 'white')]
                      )
        
        # Estilo para Botões Secundários (Voltar/Cancelar)
        self.style.configure('Secundario.TButton', font=("Helvetica", 11), padding=[15, 8])
        
        # Estilo para Frames de Conteúdo (padding interno e leve borda/fundo)
        # NOTA: O 'clam' já dá um bom visual, mas podemos adicionar padding.
        self.style.configure('Content.TFrame', padding=15)
        
        # Estilo para o Label de Sucesso
        self.style.configure('Sucesso.TLabel', font=("Helvetica", 16, "bold"), foreground="#28a745") # Verde Sucesso
        
        # Estilo para detalhes em TelaSucesso
        self.style.configure('Detalhes.TLabel', font=("Courier", 10), foreground="#0056b3") # Azul escuro para contraste

    def mostrar_tela(self, page_name):
        frame = self.frames[page_name]
        
        # Se for a tela de sucesso, atualiza os detalhes
        if page_name == "TelaSucesso":
            frame.mostrar_detalhes()
            
        frame.tkraise()

    def finalizar_agendamento(self):
        global TOTAL_ATENDIMENTOS
        TOTAL_ATENDIMENTOS += 1
        self.mostrar_tela("TelaSucesso")
        
# --- Placeholder para TelaAgendamento (Mantenha a sua lógica anterior de abrir_calendario, avançar_para_dados) ---
class TelaAgendamento(ttk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent, style='Content.TFrame')
        self.controller = controller
        # ... O restante da sua classe TelaAgendamento, refatorando os widgets para usar os novos estilos ...
        ttk.Label(self, text="Agendamento de Consulta", style='Titulo.TLabel').pack(pady=20)
        # Exemplo de label da data selecionada (precisa ser configurada na classe completa)
        self.lbl_data_selecionada = ttk.Label(self, text="Data Selecionada: Nenhuma", style='Campo.TLabel')
        self.lbl_data_selecionada.pack()
        # Exemplo de botão
        # ttk.Button(self, text="Abrir Calendário", command=self.abrir_calendario).pack(pady=5)
        # ttk.Button(self, text="Avançar", command=self.avancar_para_dados, style='Principal.TButton').pack(pady=20)
        # NOTA: Você deve integrar o conteúdo completo da sua TelaAgendamento aqui e aplicar os estilos.


# --- Tela 2: Cadastro de Dados Pessoais REVISADA ---
class TelaDadosPessoais(ttk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent, style='Content.TFrame') # Aplicando o estilo de frame
        self.controller = controller

        # Layout com padding mais generoso
        ttk.Label(self, text="Dados Pessoais do Paciente", style='Titulo.TLabel').pack(pady=(15, 30))

        campos = [("Nome:", "nome"), ("CPF:", "cpf"), ("Telefone:", "telefone"), ("Email:", "email")]
        self.entries = {}

        # Frame principal para centralizar os campos
        form_frame = ttk.Frame(self)
        form_frame.pack(padx=50, pady=10, fill='x', anchor='center')

        for label_text, key in campos:
            frame = ttk.Frame(form_frame)
            frame.pack(pady=8, fill='x')
            
            # Usando o novo estilo Campo.TLabel
            ttk.Label(frame, text=label_text, width=15, anchor="w", style='Campo.TLabel').pack(side="left", padx=5)
            entry = ttk.Entry(frame, width=40, font=("Helvetica", 11)) # Fonte ligeiramente maior
            entry.pack(side="right", expand=True, fill='x', padx=5)
            self.entries[key] = entry

        # Botões com novos estilos e alinhamento
        frame_botoes = ttk.Frame(self)
        frame_botoes.pack(pady=40)
        
        # Botão Voltar
        ttk.Button(frame_botoes, text="< Voltar", command=lambda: controller.mostrar_tela("TelaAgendamento"), 
                   style='Secundario.TButton').pack(side="left", padx=20)
        
        # Botão Principal de Confirmação
        ttk.Button(frame_botoes, text="Confirmar Agendamento >", command=self.confirmar_e_finalizar, 
                   style='Principal.TButton').pack(side="left", padx=20)

    # ... manter as funções validar_e_armazenar e confirmar_e_finalizar originais ...
    def validar_e_armazenar(self, nome, cpf, telefone, email):
        # ... (sua lógica de validação original) ...
        if not (nome and cpf and telefone and email):
            messagebox.showwarning("Erro de Validação", "Por favor, preencha todos os campos de dados pessoais.")
            return False
        
        # Validação de formato de e-mail mais robusta
        import re
        if not re.match(r"[^@]+@[^@]+\.[^@]+", email):
            messagebox.showwarning("Erro de Validação", "O formato do e-mail é inválido.")
            return False
        
        # Validação de CPF (opcional, mas recomendado para 'mais detalhes')
        if len(cpf.replace('.', '').replace('-', '')) != 11 or not cpf.replace('.', '').replace('-', '').isdigit():
             messagebox.showwarning("Erro de Validação", "O CPF deve conter 11 dígitos.")
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


# --- Tela 3: Sucesso REVISADA ---
class TelaSucesso(ttk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent, style='Content.TFrame')
        self.controller = controller
        
        # Ícone de Sucesso (apenas um símbolo para dar um toque visual)
        ttk.Label(self, text="✔", font=("Arial", 60, "bold"), foreground="#28a745").pack(pady=(20, 10))

        # Título com novo estilo de Sucesso
        ttk.Label(self, text="Agendamento Concluído!", style='Sucesso.TLabel').pack(pady=10)
        
        ttk.Label(self, text="Sua consulta foi agendada com sucesso!", font=("Arial", 12)).pack(pady=5)
        
        # Frame para os detalhes com borda e fundo leve (simulando um "card")
        detalhes_frame = ttk.Frame(self, padding="15 15 15 15", relief="groove")
        detalhes_frame.pack(pady=20, padx=50, fill='x')
        
        ttk.Label(detalhes_frame, text="Resumo do Agendamento", style='Subtitulo.TLabel').pack(anchor="w", pady=(0, 10))

        # Label para os detalhes formatados, usando o novo estilo 'Detalhes.TLabel'
        self.lbl_detalhes = ttk.Label(detalhes_frame, text="", style='Detalhes.TLabel', justify=tk.LEFT)
        self.lbl_detalhes.pack(fill='x', anchor="w")

        # Contador em um frame separado para destaque
        contador_frame = ttk.Frame(self, padding="10 5 10 5")
        contador_frame.pack(pady=15)

        self.lbl_contador = ttk.Label(contador_frame, text="", font=("Arial", 12, "italic"), foreground="#007bff") # Azul principal
        self.lbl_contador.pack()
        
        # Botão com estilo principal
        ttk.Button(self, text="Fazer Novo Agendamento", command=self.reiniciar_aplicativo, 
                   style='Principal.TButton').pack(pady=30)

    def mostrar_detalhes(self):
        # Formatação mais limpa e organizada dos detalhes para o Courier
        detalhes = f"""
NOME: {dados_pessoais.get('nome', 'N/A')}
CPF:  {dados_pessoais.get('cpf', 'N/A')}
TEL:  {dados_pessoais.get('telefone', 'N/A')}
EMAIL: {dados_pessoais.get('email', 'N/A')}
{'='*35}
CIDADE: {dados_agendamento.get('cidade', 'N/A')}
DATA:   {dados_agendamento.get('data_selecionada', 'N/A')}
HORÁRIO: {dados_agendamento.get('horario', 'N/A')}
MÉDICO: {dados_agendamento.get('medico', 'N/A')}
"""
        self.lbl_detalhes.config(text=detalhes.strip())
        self.lbl_contador.config(text=f"Você é o agendamento número: {TOTAL_ATENDIMENTOS:04d}") # Formato com 4 dígitos para visual

    def reiniciar_aplicativo(self):
        # Limpar os dados para um novo agendamento
        dados_agendamento.clear()
        dados_pessoais.clear()
        self.controller.mostrar_tela("TelaAgendamento")

# --- Inicialização do Aplicativo ---
if __name__ == "__main__":
    # Simulação de dados iniciais para testar a TelaSucesso
    dados_agendamento = {"cidade": "São Paulo", "data_selecionada": "31/12/2025", "horario": "14:00", "medico": "Dr. House"}
    dados_pessoais = {"nome": "Maria Silva", "cpf": "123.456.789-00", "telefone": "(11) 98765-4321", "email": "maria@exemplo.com"}
    TOTAL_ATENDIMENTOS = 42
    
    app = AplicativoAgendamento()
    app.mainloop()
