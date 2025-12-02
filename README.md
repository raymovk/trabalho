# trabalho
import tkinter as tk
from tkinter import messagebox
from peewee import *


db = SqliteDatabase("hotel.db")

class BaseModel(Model):
    class Meta:
        database = db

class TipoQuarto(BaseModel):
    nome = CharField()
    valor_diaria = DecimalField()

    def __str__(self):
        return f"{self.id} - {self.nome} - R$ {self.valor_diaria:.2f}"

db.connect()
db.create_tables([TipoQuarto])


def atualizar_lista():
    listbox.delete(0, tk.END)
    for tq in TipoQuarto.select().order_by(TipoQuarto.id):
        listbox.insert(tk.END, str(tq))

def limpar_campos():
    entry_nome.delete(0, tk.END)
    entry_valor.delete(0, tk.END)

def salvar():
    nome = entry_nome.get().strip()
    valor = entry_valor.get().strip()

    if not nome or not valor:
        messagebox.showerror("Erro", "Preencha todos os campos!")
        return

    try:
        valor = float(valor)
    except ValueError:
        messagebox.showerror("Erro", "Valor da diária deve ser numérico!")
        return

    global id_edicao
    if id_edicao:  # Atualização
        tq = TipoQuarto.get_by_id(id_edicao)
        tq.nome = nome
        tq.valor_diaria = valor
        tq.save()
        messagebox.showinfo("Sucesso", "Tipo de quarto atualizado com sucesso!")
        id_edicao = None
    else:  # Novo cadastro
        TipoQuarto.create(nome=nome, valor_diaria=valor)
        messagebox.showinfo("Sucesso", "Tipo de quarto cadastrado com sucesso!")

    limpar_campos()
    atualizar_lista()

def editar():
    global id_edicao
    selecao = listbox.curselection()
    if not selecao:
        messagebox.showerror("Erro", "Selecione um item para editar!")
        return
    indice = selecao[0]
    texto = listbox.get(indice)
    id_registro = int(texto.split(" - ")[0])

    tq = TipoQuarto.get_by_id(id_registro)
    entry_nome.delete(0, tk.END)
    entry_nome.insert(0, tq.nome)
    entry_valor.delete(0, tk.END)
    entry_valor.insert(0, tq.valor_diaria)

    id_edicao = tq.id

def excluir():
    selecao = listbox.curselection()
    if not selecao:
        messagebox.showerror("Erro", "Selecione um item para excluir!")
        return
    indice = selecao[0]
    texto = listbox.get(indice)
    id_registro = int(texto.split(" - ")[0])

    tq = TipoQuarto.get_by_id(id_registro)
    tq.delete_instance()
    messagebox.showinfo("Sucesso", "Registro excluído com sucesso!")
    atualizar_lista()

janela = tk.Tk()
janela.title("Cadastro de Tipo de Quarto")


id_edicao = None


tk.Label(janela, text="Nome do quarto:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
entry_nome = tk.Entry(janela, width=30)
entry_nome.grid(row=0, column=1, padx=5, pady=5)

tk.Label(janela, text="Valor da diária:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
entry_valor = tk.Entry(janela, width=30)
entry_valor.grid(row=1, column=1, padx=5, pady=5)


btn_salvar = tk.Button(janela, text="Salvar", command=salvar)
btn_salvar.grid(row=2, column=0, padx=5, pady=5)

btn_limpar = tk.Button(janela, text="Limpar", command=limpar_campos)
btn_limpar.grid(row=2, column=1, padx=5, pady=5)


listbox = tk.Listbox(janela, width=50, height=10)
listbox.grid(row=3, column=0, columnspan=2, padx=5, pady=5)


btn_editar = tk.Button(janela, text="Editar", command=editar)
btn_editar.grid(row=4, column=0, padx=5, pady=5)

btn_excluir = tk.Button(janela, text="Excluir", command=excluir)
btn_excluir.grid(row=4, column=1, padx=5, pady=5)


atualizar_lista()

janela.mainloop()
