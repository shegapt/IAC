import tkinter as tk
from tkinter import ttk, scrolledtext, messagebox
from tkcalendar import DateEntry
from datetime import datetime
import json
import os
from chatterbot import ChatBot
from chatterbot.trainers import ListTrainer

# Ficheiros de dados
FICHEIRO_MEMORIA = 'cultivation_data.json'
FICHEIRO_DADOS_UTILIZADOR = "user_data.json"

# Dados de memória inicial
memoria = {
    'data': [],
    'plantas': [],
    'ambientes': [],
    'calculos': [],
}

memoria_utilizador = {
    "plantas": [],
    "ambientes": [],
    "calculos": [],
    "dados_diarios": [],
}

dados_localizacao = {
    "cidade": "",
    "país": "",
}

# Carregar dados do ficheiro JSON
def carregar_dados(ficheiro):
    if os.path.exists(ficheiro):
        with open(ficheiro, "r") as f:
            return json.load(f)
    else:
        return None

# Guardar dados no ficheiro JSON
def guardar_dados(dados, ficheiro):
    with open(ficheiro, "w") as f:
        json.dump(dados, f, indent=4)

# Carregar memória inicial
memoria_carregada = carregar_dados(FICHEIRO_MEMORIA)
if memoria_carregada:
    memoria = memoria_carregada
else:
    guardar_dados(memoria, FICHEIRO_MEMORIA)

# Inicialização do Chatbot
bot = ChatBot(
    'Bot',
    storage_adapter='chatterbot.storage.SQLStorageAdapter',
    database_uri='sqlite:///database.sqlite3'
)

trainer = ListTrainer(bot)

# Treinamento inicial do Chatbot
trainer.train([
    'Olá',
    'Olá, em que posso ajudar?',
    'Como estás?',
    'Estou bem, obrigado. E tu?',
    'Estou ótimo!',
    'Preciso de dicas para o cultivo de cannabis',
    'Claro, tens alguma estirpe específica em mente?',
    'Sim, estou interessado numa estirpe Indica',
    'Ótimo! Qual é a temperatura e humidade do teu ambiente?',
    'A temperatura está por volta dos 25°C e a humidade está a 60%',
    'Essas condições são boas para Indica. Certifica-te de manter uma boa ventilação.',
    'Obrigado pelas dicas.',
    'De nada! Lembra-te de monitorizar a tua planta regularmente.'
])

# Carregar dados do utilizador do ficheiro JSON
def carregar_dados_utilizador(ficheiro):
    if os.path.exists(ficheiro):
        with open(ficheiro, "r") as f:
            return json.load(f)
    else:
        return None

# Guardar dados do utilizador no ficheiro JSON
def guardar_dados_utilizador(dados, ficheiro):
    with open(ficheiro, "w") as f:
        json.dump(dados, f, indent=4)

# Carregar memória inicial do utilizador
dados_utilizador_carregados = carregar_dados_utilizador(FICHEIRO_DADOS_UTILIZADOR)
if dados_utilizador_carregados:
    memoria_utilizador = dados_utilizador_carregados
else:
    guardar_dados_utilizador(memoria_utilizador, FICHEIRO_DADOS_UTILIZADOR)

# Função para recolher dados diários
def recolher_dados(root, planta_selecionada):
    janela_recolha = tk.Toplevel(root)
    janela_recolha.title("Adicionar Dados Diários")
    janela_recolha.geometry("600x500")

    tk.Label(janela_recolha, text="Data (DD-MM-AAAA):").grid(row=0, column=0, padx=10, pady=10)
    entrada_data = DateEntry(janela_recolha, width=12, background='darkblue', foreground='white', borderwidth=2)
    entrada_data.grid(row=0, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="Hora (HH:MM):").grid(row=1, column=0, padx=10, pady=10)
    entrada_hora = tk.Entry(janela_recolha)
    entrada_hora.grid(row=1, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="Adição de Água:").grid(row=2, column=0, padx=10, pady=10)
    check_agua = tk.Checkbutton(janela_recolha)
    check_agua.grid(row=2, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="Adição de Nutrientes:").grid(row=3, column=0, padx=10, pady=10)
    check_nutrientes = tk.Checkbutton(janela_recolha)
    check_nutrientes.grid(row=3, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="Adição de Inseticida:").grid(row=4, column=0, padx=10, pady=10)
    entrada_inseticida = tk.Entry(janela_recolha)
    entrada_inseticida.grid(row=4, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="Tamanho da Planta:").grid(row=5, column=0, padx=10, pady=10)
    entrada_tamanho = tk.Entry(janela_recolha)
    entrada_tamanho.grid(row=5, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="PPM:").grid(row=6, column=0, padx=10, pady=10)
    entrada_ppm = tk.Entry(janela_recolha)
    entrada_ppm.grid(row=6, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="EC:").grid(row=7, column=0, padx=10, pady=10)
    entrada_ec = tk.Entry(janela_recolha)
    entrada_ec.grid(row=7, column=1, padx=10, pady=10)

    tk.Label(janela_recolha, text="pH:").grid(row=8, column=0, padx=10, pady=10)
    entrada_ph = tk.Entry(janela_recolha)
    entrada_ph.grid(row=8, column=1, padx=10, pady=10)

    def guardar_dados_diarios():
        data = entrada_data.get_date().strftime('%d-%m-%Y') if entrada_data.get_date() else ""
        hora = entrada_hora.get()
        agua = check_agua.instate(['selected'])
        nutrientes = check_nutrientes.instate(['selected'])
        inseticida = entrada_inseticida.get()
        tamanho = entrada_tamanho.get()
        ppm = entrada_ppm.get()
        ec = entrada_ec.get()
        ph = entrada_ph.get()

        if not data or not hora:
            messagebox.showerror("Erro", "Por favor, preencha a data e a hora.")
            return

        if not agua and not nutrientes:
            messagebox.showerror("Erro", "Selecione pelo menos Adição de Água ou Adição de Nutrientes.")
            return

        entrada_diaria = {
            "data": data,
            "hora": hora,
            "agua": agua,
            "nutrientes": nutrientes,
            "inseticida": inseticida,
            "tamanho": tamanho,
            "ppm": ppm,
            "ec": ec,
            "ph": ph,
        }

        for planta in memoria_utilizador["plantas"]:
            if planta["nome"] == planta_selecionada:
                planta["dados_diarios"].append(entrada_diaria)
                break

        guardar_dados_utilizador(memoria_utilizador, FICHEIRO_DADOS_UTILIZADOR)
        messagebox.showinfo("Guardado", "Dados diários guardados com sucesso!")
        janela_recolha.destroy()

    tk.Button(janela_recolha, text="Guardar", command=guardar_dados_diarios).grid(row=9, column=0, columnspan=2, pady=20)

# Função para visualizar dados diários
def visualizar_dados_diarios(root, planta_selecionada):
    janela_visualizacao = tk.Toplevel(root)
    janela_visualizacao.title(f"Dados Diários de {planta_selecionada}")
    janela_visualizacao.geometry("800x600")

    # Obtém os dados da planta selecionada
    dados_planta = None
    for planta in memoria_utilizador["plantas"]:
        if planta["nome"] == planta_selecionada:
            dados_planta = planta
            break

    if not dados_planta or "dados_diarios" not in dados_planta or not dados_planta["dados_diarios"]:
        tk.Label(janela_visualizacao, text="Não há dados diários disponíveis para esta planta.").pack(pady=20)
        return

    # Criação da tabela para exibir os dados diários
    tree = ttk.Treeview(janela_visualizacao)
    tree["columns"] = ("Data", "Hora", "Água", "Nutrientes", "Inseticida", "Tamanho", "PPM", "EC", "pH")
    tree.column("#0", width=0, stretch=tk.NO)
    tree.column("Data", anchor=tk.CENTER, width=100)
    tree.column("Hora", anchor=tk.CENTER, width=100)
    tree.column("Água", anchor=tk.CENTER, width=80)
    tree.column("Nutrientes", anchor=tk.CENTER, width=100)
    tree.column("Inseticida", anchor=tk.CENTER, width=100)
    tree.column("Tamanho", anchor=tk.CENTER, width=80)
    tree.column("PPM", anchor=tk.CENTER, width=80)
    tree.column("EC", anchor=tk.CENTER, width=80)
    tree.column("pH", anchor=tk.CENTER, width=80)

    tree.heading("#0", text="", anchor=tk.CENTER)
    tree.heading("Data", text="Data", anchor=tk.CENTER)
    tree.heading("Hora", text="Hora", anchor=tk.CENTER)
    tree.heading("Água", text="Água", anchor=tk.CENTER)
    tree.heading("Nutrientes", text="Nutrientes", anchor=tk.CENTER)
    tree.heading("Inseticida", text="Inseticida", anchor=tk.CENTER)
    tree.heading("Tamanho", text="Tamanho", anchor=tk.CENTER)
    tree.heading("PPM", text="PPM", anchor=tk.CENTER)
    tree.heading("EC", text="EC", anchor=tk.CENTER)
    tree.heading("pH", text="pH", anchor=tk.CENTER)

    for dados in dados_planta["dados_diarios"]:
        tree.insert("", tk.END, values=(
            dados["data"],
            dados["hora"],
            "Sim" if dados["agua"] else "Não",
            "Sim" if dados["nutrientes"] else "Não",
            dados["inseticida"],
            dados["tamanho"],
            dados["ppm"],
            dados["ec"],
            dados["ph"]
        ))

    tree.pack(pady=20, padx=20, fill=tk.BOTH, expand=True)

# Função principal
def main():
    root = tk.Tk()
    root.title("Cannabis Cultivation Assistant")
    root.geometry("800x600")

    tk.Label(root, text="Bem-vindo ao Assistente de Cultivo de Cannabis", font=("Helvetica", 16)).pack(pady=20)

    # Função para iniciar um novo cultivo
    def iniciar_novo_cultivo():
        nome_planta = entry_nome_planta.get()

        if not nome_planta:
            messagebox.showerror("Erro", "Por favor, introduza o nome da planta.")
            return

        for planta in memoria_utilizador["plantas"]:
            if planta["nome"].lower() == nome_planta.lower():
                messagebox.showerror("Erro", f"A planta '{nome_planta}' já está na sua lista.")
                return

        memoria_utilizador["plantas"].append({
            "nome": nome_planta,
            "dados_diarios": []
        })

        guardar_dados_utilizador(memoria_utilizador, FICHEIRO_DADOS_UTILIZADOR)
        messagebox.showinfo("Sucesso", f"O cultivo da planta '{nome_planta}' foi iniciado com sucesso.")
        janela_novo_cultivo.destroy()

    # Interface para iniciar um novo cultivo
    def interface_novo_cultivo():
        global janela_novo_cultivo
        janela_novo_cultivo = tk.Toplevel(root)
        janela_novo_cultivo.title("Novo Cultivo")
        janela_novo_cultivo.geometry("400x200")

        tk.Label(janela_novo_cultivo, text="Nome da Planta:").grid(row=0, column=0, padx=10, pady=10)
        global entry_nome_planta
        entry_nome_planta = tk.Entry(janela_novo_cultivo)
        entry_nome_planta.grid(row=0, column=1, padx=10, pady=10)

        tk.Button(janela_novo_cultivo, text="Iniciar Cultivo", command=iniciar_novo_cultivo).grid(row=1, column=0, columnspan=2, pady=20)

    # Função para selecionar uma planta e exibir seus dados diários
    def selecionar_planta():
        planta_selecionada = combo_plantas.get()

        if not planta_selecionada:
            messagebox.showerror("Erro", "Por favor, selecione uma planta.")
            return

        visualizar_dados_diarios(root, planta_selecionada)

    # Carrega as plantas do utilizador
    plantas_utilizador = [planta["nome"] for planta in memoria_utilizador["plantas"]]

    # Interface principal
    tk.Label(root, text="Selecione uma Planta:", font=("Helvetica", 14)).pack(pady=10)

    global combo_plantas
    combo_plantas = ttk.Combobox(root, values=plantas_utilizador)
    combo_plantas.pack(pady=10)

    tk.Button(root, text="Adicionar Nova Planta", command=interface_novo_cultivo).pack(pady=10)
    tk.Button(root, text="Visualizar Dados Diários", command=selecionar_planta).pack(pady=10)

    root.mainloop()

if __name__ == "__main__":
    main()
