import tkinter as tk
from tkinter import ttk, messagebox
import mysql.connector

# Configurações do banco de dados
db_config = {
    "host": "localhost",
    "user": "root",
    "password": "99751422aA@",
    "database": "cadastro_de_cliente",
    "port": 3306
}

# Função para obter o próximo número do pedido
def obter_proximo_numero_pedido():
    try:
        conexao = mysql.connector.connect(**db_config)
        cursor = conexao.cursor()
        cursor.execute("SELECT MAX(numero) FROM pedidos")  # Supondo que exista uma tabela 'pedidos'
        resultado = cursor.fetchone()
        return (resultado[0] + 1) if resultado[0] is not None else 1  # Retorna 1 se não houver pedidos
    except mysql.connector.Error as err:
        messagebox.showerror("Erro", f"Erro ao obter próximo número do pedido: {err}")
        return 1  # Retorna 1 em caso de erro
    finally:
        if cursor:
            cursor.close()
        if conexao:
            conexao.close()

# Função para alternar a visibilidade do frame de cadastro
def alternar_cadastro():
    if frame_cadastro.winfo_viewable():  # Verifica se o frame está visível
        frame_cadastro.pack_forget()      # Oculta o frame se estiver visível
    else:
        frame_pedido.pack_forget()        # Oculta o frame de pedido, se estiver visível
        frame_cadastro.pack(fill="both", expand=True)  # Exibe o frame de cadastro

# Função para alternar a visibilidade do frame de pedido
def alternar_pedido():
    if frame_pedido.winfo_viewable():  # Verifica se o frame está visível
        frame_pedido.pack_forget()      # Oculta o frame se estiver visível
    else:
        frame_cadastro.pack_forget()    # Oculta o frame de cadastro, se estiver visível
        entrada_pedido.delete(0, tk.END)  # Limpa o campo de entrada
        numero_pedido = obter_proximo_numero_pedido()  # Obtém o próximo número do pedido
        entrada_pedido.insert(0, numero_pedido)  # Preenche o campo com o número do pedido
        frame_pedido.pack(fill="both", expand=True)  # Exibe o frame de pedido

# Função para salvar o cadastro
def salvar_cadastro():
    dados = {
        "nome": entrada_nome.get(),
        "ddd": entrada_ddd.get(),
        "numero": entrada_numero.get(),
        "endereco": entrada_endereco.get(),
        "cidade": entrada_cidade.get(),
        "estado": combobox_estado.get()
    }

    # Conectar ao banco de dados e inserir os dados
    try:
        conexao = mysql.connector.connect(**db_config)
        cursor = conexao.cursor()
        sql = "INSERT INTO cadastro (nome, ddd, numero, endereco, cidade, estado) VALUES (%s, %s, %s, %s, %s, %s)"
        valores = (
            dados["nome"], dados["ddd"], dados["numero"], dados["endereco"], dados["cidade"], dados["estado"]
        )
        cursor.execute(sql, valores)
        conexao.commit()
        messagebox.showinfo("Sucesso", "Cadastro salvo com sucesso!")
    except mysql.connector.Error as err:
        messagebox.showerror("Erro", f"Erro ao salvar cadastro: {err}")
    finally:
        if cursor:
            cursor.close()
        if conexao:
            conexao.close()

# Janela principal
Janela = tk.Tk()
Janela.title("Robrac")
largura_tela = Janela.winfo_screenwidth()
altura_tela = Janela.winfo_screenheight()
Janela.geometry(f"{largura_tela}x{altura_tela}")
Janela.configure(bg='#F0F0F0')

# Frame para os botões no topo
frame_botoes = tk.Frame(Janela, bg='#F0F0F0')
frame_botoes.pack(anchor="nw", fill="x", padx=5, pady=5)

# Botão para alternar o frame de cadastro
botao_cadastro = tk.Button(frame_botoes, text="Cadastro", bg="#0000ff", fg="white", height=1, width=10, font=("Roboto", 11),
                           command=alternar_cadastro)
botao_cadastro.pack(side=tk.LEFT, padx=5, pady=5)

# Botão para alternar o frame de pedido
botao_criar_pedido = tk.Button(frame_botoes, text="Criar Pedido", bg="#000080", fg="white", height=1, width=10, font=("Roboto", 11),
                               command=alternar_pedido)
botao_criar_pedido.pack(side=tk.LEFT, padx=5, pady=5)

# Frame de Cadastro (inicialmente oculto)
frame_cadastro = tk.Frame(Janela, bg='#F0F0F0')

label_nome = tk.Label(frame_cadastro, text="Nome:")
label_nome.grid(row=0, column=0, padx=5, pady=5, sticky="w")
entrada_nome = tk.Entry(frame_cadastro, width=30)
entrada_nome.grid(row=0, column=1, padx=5, pady=5, sticky="w")

label_ddd = tk.Label(frame_cadastro, text="DDD:")
label_ddd.grid(row=1, column=0, padx=5, pady=5, sticky="w")
entrada_ddd = tk.Entry(frame_cadastro, width=5)
entrada_ddd.grid(row=1, column=1, padx=5, pady=5, sticky="w")

label_numero = tk.Label(frame_cadastro, text="Número:")
label_numero.grid(row=2, column=0, padx=5, pady=5, sticky="w")
entrada_numero = tk.Entry(frame_cadastro, width=20)
entrada_numero.grid(row=2, column=1, padx=5, pady=5, sticky="w")

label_endereco = tk.Label(frame_cadastro, text="Endereço:")
label_endereco.grid(row=3, column=0, padx=5, pady=5, sticky="w")
entrada_endereco = tk.Entry(frame_cadastro, width=30)
entrada_endereco.grid(row=3, column=1, padx=5, pady=5, sticky="w")

label_cidade = tk.Label(frame_cadastro, text="Cidade:")
label_cidade.grid(row=4, column=0, padx=5, pady=5, sticky="w")
entrada_cidade = tk.Entry(frame_cadastro, width=30)
entrada_cidade.grid(row=4, column=1, padx=5, pady=5, sticky="w")

label_estado = tk.Label(frame_cadastro, text="Estado:")
label_estado.grid(row=5, column=0, padx=5, pady=5, sticky="w")
estados = ["AC", "AL", "AP", "AM", "BA", "CE", "DF", "ES", "GO", "MA", "MT", "MS", "MG", "PA", "PB", "PR", "PE", "PI", "RJ", "RN", "RS", "RO", "RR", "SC", "SP", "SE", "TO"]
combobox_estado = ttk.Combobox(frame_cadastro, values=estados, width=5)
combobox_estado.grid(row=5, column=1, padx=5, pady=5, sticky="w")

botao_salvar = tk.Button(frame_cadastro, text="Salvar Cadastro", command=salvar_cadastro)
botao_salvar.grid(row=6, column=1, padx=5, pady=10, sticky="w")

# Frame de Pedido (inicialmente oculto)
frame_pedido = tk.Frame(Janela, bg='#F0F0F0')
label_pedido = tk.Label(frame_pedido, text="Número do Pedido:")
label_pedido.grid(row=0, column=0, padx=5, pady=5, sticky="w")
entrada_pedido = tk.Entry(frame_pedido, width=30)
entrada_pedido.grid(row=0, column=1, padx=5, pady=5, sticky="w")

label_cliente = tk.Label(frame_pedido, text="Nome do Cliente:")
label_cliente.grid(row=1, column=0, padx=5, pady=5, sticky="w")
combobox_cliente = ttk.Combobox(frame_pedido, values=[], width=30)
combobox_cliente.grid(row=1, column=1, padx=5, pady=5, sticky="w")

botao_cadastrar_cliente = tk.Button(frame_pedido, text="Cadastrar Cliente", command=alternar_cadastro)
botao_cadastrar_cliente.grid(row=2, column=1, padx=5, pady=5, sticky="w")

# Inicia o aplicativo sem frames visíveis
Janela.mainloop()
