# Efici-ncia-Operacional-
Relatório de eficiência com gráfico com real-time update. Criado para der intuitivo e para pessoas que querem medir sua eficiência, mas não tenho conhecimento em Data, como power bi por exemplo.

import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import ipywidgets as widgets
from IPython.display import display, clear_output
import seaborn as sns

# --- DADOS DAS TAREFAS ---
# Tarefas financeiras e administrativas para André e Pedro
atividades = {
    'Segunda': [
        {"tarefa": "Reunião de alinhamento financeiro (André e Pedro)", "concluida": False},
        {"tarefa": "Fechamento do fluxo de caixa semanal (André)", "concluida": False},
        {"tarefa": "Pagamento de fornecedores (André)", "concluida": False},
        {"tarefa": "Análise de custos de produção (Pedro)", "concluida": False}
    ],
    'Terça': [
        {"tarefa": "Conciliação bancária (André)", "concluida": False},
        {"tarefa": "Emissão de notas fiscais (André)", "concluida": False},
        {"tarefa": "Controle de estoque de matérias-primas (Pedro)", "concluida": False},
        {"tarefa": "Revisão de contratos com clientes (Pedro)", "concluida": False}
    ],
    'Quarta': [
        {"tarefa": "Relatório de contas a receber (André)", "concluida": False},
        {"tarefa": "Reunião com contabilidade (André e Pedro)", "concluida": False},
        {"tarefa": "Análise de viabilidade de novos projetos (Pedro)", "concluida": False},
        {"tarefa": "Aprovação de despesas (Pedro)", "concluida": False}
    ],
    'Quinta': [
        {"tarefa": "Controle de folha de pagamento (André)", "concluida": False},
        {"tarefa": "Atualização de planilhas financeiras (André)", "concluida": False},
        {"tarefa": "Planejamento de investimentos (Pedro)", "concluida": False},
        {"tarefa": "Negociação com fornecedores (Pedro)", "concluida": False}
    ],
    'Sexta': [
        {"tarefa": "Relatório financeiro semanal (André)", "concluida": False},
        {"tarefa": "Encerramento fiscal parcial (André)", "concluida": False},
        {"tarefa": "Revisão de metas financeiras (Pedro)", "concluida": False},
        {"tarefa": "Planejamento orçamentário (Pedro)", "concluida": False}
    ]
}

# --- CRIAÇÃO DOS WIDGETS DA INTERFACE ---

# 1. Título do Painel (com HTML para estilo)
title_html = widgets.HTML(
    value="""
    <style>
        .title-container {
            background-color: #f0f8ff;
            border-left: 5px solid #4682b4;
            padding: 10px 20px;
            margin-bottom: 20px;
        }
    </style>
    <div class="title-container">
        <h1>📊 Painel de Produtividade Financeira</h1>
        <p><b>Responsáveis:</b> André (Administrativo) e Pedro (Diretor de Operações)</p>
        <p><b>Metas:</b> 4 tarefas por dia | 20 tarefas por semana</p>
    </div>
    """
)

# 2. Checkboxes e Acordeão para organizar as tarefas
checkboxes = {}
accordion_children = []

for dia, tarefas in atividades.items():
    checkboxes_dia = []
    for i, tarefa in enumerate(tarefas):
        cb = widgets.Checkbox(
            value=False,
            description=tarefa["tarefa"],
            indent=False,
            layout={'width': 'max-content'}
        )
        checkboxes_dia.append(cb)
    
    # Agrupa os checkboxes de um dia em uma caixa vertical
    vbox = widgets.VBox(children=checkboxes_dia)
    accordion_children.append(vbox)
    checkboxes[dia] = checkboxes_dia

accordion = widgets.Accordion(children=accordion_children)
for i, dia in enumerate(atividades.keys()):
    accordion.set_title(i, f"🗓️ {dia}")

# 3. Widgets de Saída para o gráfico e o resumo
output_plot = widgets.Output()
summary_html = widgets.HTML()


# --- FUNÇÕES DE LÓGICA E ATUALIZAÇÃO ---

def calcular_progresso():
    """Calcula o progresso diário e total."""
    progresso_diario = {}
    total_tarefas = 0
    concluidas = 0
    
    for dia in atividades.keys():
        concluidas_dia = sum(1 for cb in checkboxes[dia] if cb.value)
        total_dia = len(atividades[dia])
        progresso_diario[dia] = (concluidas_dia, total_dia)
        concluidas += concluidas_dia
        total_tarefas += total_dia
    
    return progresso_diario, concluidas, total_tarefas

def update_interface(change=None):
    """Função chamada para redesenhar o gráfico e o resumo."""
    progresso_diario, total_concluidas, total_tarefas = calcular_progresso()
    progresso_percentual = (total_concluidas / total_tarefas) * 100 if total_tarefas > 0 else 0

    # Atualiza o gráfico no widget de saída
    with output_plot:
        clear_output(wait=True)
        sns.set_theme(style="whitegrid", palette="pastel")
        fig, ax = plt.subplots(figsize=(12, 6))
        
        dias = list(progresso_diario.keys())
        concluidas = [p[0] for p in progresso_diario.values()]
        totais = [p[1] for p in progresso_diario.values()]
        pendentes = [totais[i] - concluidas[i] for i in range(len(dias))]
        
        # Barras empilhadas
        ax.bar(dias, concluidas, color='mediumseagreen', label='Concluídas')
        ax.bar(dias, pendentes, bottom=concluidas, color='lightcoral', label='Pendentes')
        
        # Linha de progresso acumulado
        acumulado = np.cumsum(concluidas) / total_tarefas * 100
        ax2 = ax.twinx()
        ax2.plot(dias, acumulado, 'o-', color='steelblue', linewidth=3, markersize=8, label='Progresso Semanal')

        # Personalização do Gráfico
        ax.set_title('Desempenho Diário e Progresso Semanal', fontsize=18, pad=20, weight='bold')
        ax.set_ylabel('Tarefas por Dia', fontsize=12)
        ax.set_ylim(0, max(totais) + 1)
        ax.legend(loc='upper left')
        
        for i, (c, t) in enumerate(zip(concluidas, totais)):
            if c > 0:
                ax.text(i, c/2, f"{c}/{t}", ha='center', va='center', color='white', weight='bold')

        ax2.set_ylabel('Progresso Semanal (%)', fontsize=12)
        ax2.set_ylim(0, 110)
        ax2.yaxis.set_major_formatter(plt.FuncFormatter(lambda x, _: f'{x:.0f}%'))
        ax2.grid(False)
        ax2.legend(loc='upper right')
        
        plt.tight_layout()
        plt.show()

    # Mensagem motivacional
    if progresso_percentual == 100:
        msg = ("<div style='color: green; font-weight: bold;'>✅ Excelente! Todas as metas financeiras foram alcançadas!</div>")
    elif progresso_percentual >= 75:
        msg = ("<div style='color: darkorange; font-weight: bold;'>🚀 Ótimo avanço! Estamos quase lá!</div>")
    elif progresso_percentual >= 50:
        msg = ("<div style='color: #4682b4; font-weight: bold;'>📊 Bom trabalho! Continue assim para alcançar a meta.</div>")
    else:
        msg = ("<div style='color: red; font-weight: bold;'>⚠️ Atenção: O desempenho está abaixo do esperado. Foco total!</div>")

    # Atualiza o resumo em HTML
    summary_html.value = f"""
    <hr>
    <h2>🏆 Resumo Semanal</h2>
    <p><b>Progresso Total:</b> {progresso_percentual:.1f}%</p>
    <progress value='{total_concluidas}' max='{total_tarefas}' style='width: 100%; height: 25px;'></progress>
    <p><b>Tarefas Concluídas:</b> {total_concluidas} de {total_tarefas}</p>
    {msg}
    """

# --- EXECUÇÃO E EXIBIÇÃO ---

# Registra a função de atualização para ser chamada quando um checkbox mudar
for dia in checkboxes:
    for cb in checkboxes[dia]:
        cb.observe(update_interface, names='value')

# Monta o layout final da interface
app_layout = widgets.VBox([
    title_html,
    accordion,
    output_plot,
    summary_html
])

# Exibe o painel e faz a primeira chamada para renderizar o estado inicial
display(app_layout)
update_interface()