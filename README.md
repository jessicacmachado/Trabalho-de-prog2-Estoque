
# ==============================================================
# BLOCO 1: DADOS INICIAIS — LISTA DE REAGENTES
# ==============================================================
reagentes_basicos = [
    "1. Ácido Clorídrico (HCl)",
    "2. Ácido Sulfúrico (H2SO4)",
    "3. Ácido Nítrico (HNO3)",
    "4. Ácido Acético (CH3COOH)",
    "5. Hidróxido de Sódio (NaOH)",
    "6. Hidróxido de Amônio (NH4OH)",
    "7. Hidróxido de Cálcio (Ca(OH)2)",
    "8. Bicarbonato de Sódio (NaHCO3)",
    "9. Cloreto de Sódio (NaCl)",
    "10. Sulfato de Cobre II (CuSO4)",
    "11. Nitrato de Prata (AgNO3)",
    "12. Cloreto de Bário (BaCl2)",
    "13. Carbonato de Sódio (Na2CO3)",
    "14. Permanganato de Potássio (KMnO4)",
    "15. Iodeto de Potássio (KI)",
    "16. Cromato de Potássio (K2CrO4)",
    "17. Água Destilada/Deionizada (H2O)",
    "18. Álcool Etílico/Etanol (C2H5OH)",
    "19. Acetona (CH3COCH3)",
    "20. Hexano (C6H14)",
    "21. Fenolftaleína",
    "22. Metiloranje (Alaranjado de Metila)",
    "23. Azul de Bromotimol",
    "24. Solução de Amido",
    "25. Peróxido de Hidrogênio (H2O2)",
    "26. Glicerina/Glicerol",
    "27. Ácido Ascórbico",
    "28. Sulfato de Magnésio (MgSO4) Anidro",
    "29. Glicose (C6H12O6)",
    "30. Nitrato de Amônio (NH4NO3)"
]

# ==============================================================
# BLOCO 2: INICIALIZAÇÃO DO ESTOQUE
# ==============================================================
estoque = dict()
for item in reagentes_basicos:
    estoque[item] = 2.0

# ==============================================================
# INTERFACE GRÁFICA INTERATIVA (TKINTER) - 700 PIXELS
# ==============================================================

def atualizar_tabela(filtro=""):
    """Atualiza as linhas da tabela aplicando ou não o filtro de busca"""
    for row in tabela.get_children():
        tabela.delete(row)
        
    for nome, qtd in estoque.items():
        if filtro.lower() in nome.lower():
            qtd_formatada = int(qtd) if qtd.is_integer() else qtd
            tabela.insert("", tk.END, values=(nome, qtd_formatada))
            
    lbl_total.config(text=f"Total de reagentes cadastrados: {len(estoque)}")

def executar_busca():
    """Filtra o estoque dinamicamente pelo campo de texto"""
    termo = txt_busca.get()
    atualizar_tabela(termo)

def limpar_busca():
    """Reseta o campo de pesquisa e exibe a lista completa"""
    txt_busca.delete(0, tk.END)
    atualizar_tabela()

def obter_item_selecionado():
    """Auxiliar para coletar o reagente clicado na interface"""
    selecao = tabela.selection()
    if not selecao:
        messagebox.showwarning("Atenção", "Por favor, selecione um reagente na tabela ao lado!")
        return None
    # Retorna o nome do reagente que está na primeira coluna da linha selecionada
    return tabela.item(selecao, "values")[0]

def realizar_retirada():
    """Reduz quantidade do reagente com validações estruturadas"""
    reagente = obter_item_selecionado()
    if not reagente:
        return
        
    try:
        qtd = float(txt_qtd.get())
        if qtd <= 0:
            raise ValueError
    except ValueError:
        messagebox.showerror("Erro", "Digite uma quantidade válida e maior que zero!")
        return

    if estoque[reagente] >= qtd:
        estoque[reagente] -= qtd
        atualizar_tabela(txt_busca.get())
        txt_qtd.delete(0, tk.END)
        messagebox.showinfo("Sucesso", f"Retirada concluída!\nSaldo atual de {reagente}: {estoque[reagente]}")
    else:
        messagebox.showerror("Erro", f"Estoque insuficiente!\nDisponível: {estoque[reagente]}")

def realizar_devolucao():
    """Adiciona quantidade informada ao estoque do reagente selecionado"""
    reagente = obter_item_selecionado()
    if not reagente:
        return
        
    try:
        qtd = float(txt_qtd.get())
        if qtd <= 0:
            raise ValueError
    except ValueError:
        messagebox.showerror("Erro", "Digite uma quantidade válida e maior que zero!")
        return

    estoque[reagente] += qtd
    atualizar_tabela(txt_busca.get())
    txt_qtd.delete(0, tk.END)
    messagebox.showinfo("Sucesso", f"Devolução/Reposição concluída!\nNovo saldo de {reagente}: {estoque[reagente]}")

def cadastrar_novo_reagente():
    """Inclui novos itens mapeando duplicatas reais"""
    nome_novo = txt_novo_nome.get().strip()
    
    if not nome_novo:
        messagebox.showwarning("Atenção", "O nome do reagente não pode estar em branco!")
        return
        
    for existente in estoque:
        if nome_novo.lower() in existente.lower():
            messagebox.showwarning("Aviso", f"Já existe um reagente parecido cadastrado:\n{existente}")
            return
            
    proximo_numero = len(estoque) + 1
    nome_formatado = f"{proximo_numero}. {nome_novo}"
    
    estoque[nome_formatado] = 2.0
    txt_novo_nome.delete(0, tk.END)
    atualizar_tabela()
    messagebox.showinfo("Sucesso", f"'{nome_formatado}' cadastrado com 2 unidades!")

# Janela Base
janela = tk.Tk()
janela.title("LabInv - Gestão Integrada de Estoque")
janela.geometry("700x530")
janela.resizable(False, False)

# --- Cabeçalho ---
lbl_titulo = tk.Label(janela, text="🔬 LabInv - Controle de Reagentes 🧪", font=("Arial", 16, "bold"), pady=10)
lbl_titulo.pack()

# Container Principal
container_principal = tk.Frame(janela)
container_principal.pack(fill=tk.BOTH, expand=True, padx=15, pady=5)

# --- PAINEL ESQUERDA (TABELA) ---
# CORREÇÃO: Removido 'padding' e adicionado 'padx/pady' internos
frame_esquerda = tk.LabelFrame(container_principal, text=" Inventário Disponível ", font=("Arial", 10, "bold"), padx=5, pady=5)
frame_esquerda.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 10))

frame_busca_interna = tk.Frame(frame_esquerda)
frame_busca_interna.pack(fill=tk.X, pady=5)

txt_busca = ttk.Entry(frame_busca_interna, font=("Arial", 10))
txt_busca.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=(0, 5))
txt_busca.bind("<KeyRelease>", lambda event: executar_busca())

btn_limpar = ttk.Button(frame_busca_interna, text="Limpar", command=limpar_busca)
btn_limpar.pack(side=tk.RIGHT)

tabela = ttk.Treeview(frame_esquerda, columns=("Reagente", "Qtd"), show="headings", selectmode="browse")
tabela.heading("Reagente", text="Reagente / Fórmula Química")
tabela.heading("Qtd", text="Qtd")
tabela.column("Reagente", width=300, anchor=tk.W)
tabela.column("Qtd", width=50, anchor=tk.CENTER)

scroll_tabela = ttk.Scrollbar(frame_esquerda, orient=tk.VERTICAL, command=tabela.yview)
tabela.configure(yscrollcommand=scroll_tabela.set)

scroll_tabela.pack(side=tk.RIGHT, fill=tk.Y)
tabela.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

# --- PAINEL DIREITA (AÇÕES) ---
frame_direita = tk.Frame(container_principal)
frame_direita.pack(side=tk.RIGHT, fill=tk.BOTH, padx=(5, 0))

# Movimentação
# CORREÇÃO: Removido 'padding' e adicionado 'padx/pady' internos
frame_movimentacao = tk.LabelFrame(frame_direita, text=" Movimentação de Saldo ", font=("Arial", 10, "bold"), padx=10, pady=10)
frame_movimentacao.pack(fill=tk.X, pady=(0, 15))

lbl_instrucao = tk.Label(frame_movimentacao, text="Selecione um item na lista ao lado e defina a qtd:", font=("Arial", 9, "italic"), justify=tk.LEFT)
lbl_instrucao.pack(anchor=tk.W, pady=(0, 5))

txt_qtd = ttk.Entry(frame_movimentacao, font=("Arial", 11), justify=tk.CENTER)
txt_qtd.pack(fill=tk.X, pady=5)

btn_retirar = tk.Button(frame_movimentacao, text="⬇ Retirar do Estoque", bg="#ffcccc", fg="#990000", font=("Arial", 10, "bold"), command=realizar_retirada)
btn_retirar.pack(fill=tk.X, pady=3)

btn_devolver = tk.Button(frame_movimentacao, text="⬆ Devolver / Repor", bg="#ccffcc", fg="#006600", font=("Arial", 10, "bold"), command=realizar_devolucao)
btn_devolver.pack(fill=tk.X, pady=3)

# Cadastro
# CORREÇÃO: adicionado 'padx/pady' internos
frame_cadastro = tk.LabelFrame(frame_direita, text=" Cadastrar Novo Reagente ", font=("Arial", 10, "bold"), padx=10, pady=10)
frame_cadastro.pack(fill=tk.X)

lbl_novo_nome = tk.Label(frame_cadastro, text="Nome e Fórmula Química:")
lbl_novo_nome.pack(anchor=tk.W)

txt_novo_nome = ttk.Entry(frame_cadastro, font=("Arial", 10))
txt_novo_nome.pack(fill=tk.X, pady=5)

btn_cadastrar = ttk.Button(frame_cadastro, text="➕ Adicionar Novo", command=cadastrar_novo_reagente)
btn_cadastrar.pack(fill=tk.X, pady=5)

# --- Rodapé ---
lbl_total = tk.Label(janela, text="", font=("Arial", 9, "italic"), pady=5)
lbl_total.pack()

atualizar_tabela()
janela.mainloop()
# Trabalho-de-prog2-Estoque
Trabalho de programação2
