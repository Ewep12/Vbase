import sys
import os
import sqlite3
import matplotlib.pyplot as plt
from datetime import datetime
from google.cloud import bigquery
from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout, QLabel,
    QPushButton, QDateEdit, QLineEdit, QMessageBox, QFrame, QStackedLayout, QTextEdit, QTabWidget
)
from PyQt5.QtCore import QDate, Qt
from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg as FigureCanvas
from matplotlib.figure import Figure
from PyQt5.QtGui import QPalette, QColor
from collections import Counter

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = r"credenciais.json"

def inicializar_banco():
    conexao = sqlite3.connect('usuarios.db')
    cursor = conexao.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS usuarios (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nome TEXT NOT NULL,
            sobrenome TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE,
            senha TEXT NOT NULL
        )
    ''')
    conexao.commit()
    conexao.close()

inicializar_banco()

class AuthWidget(QWidget):
    def __init__(self, login_callback):
        super().__init__()
        self.login_callback = login_callback
        self.init_ui()

    def init_ui(self):
        self.layout = QVBoxLayout()
        self.setLayout(self.layout)
        self.mostrar_tela_login()

    def mostrar_tela_login(self):
        self.limpar_layout()

        self.usuario_input = QLineEdit()
        self.usuario_input.setPlaceholderText("Digite seu e-mail")

        self.senha_input = QLineEdit()
        self.senha_input.setPlaceholderText("Digite sua senha")
        self.senha_input.setEchoMode(QLineEdit.Password)

        btn_login = QPushButton("Entrar")
        btn_login.setCursor(Qt.PointingHandCursor)
        btn_login.clicked.connect(self.validar_login)

        btn_cadastrar = QPushButton("Cadastrar")
        btn_cadastrar.setCursor(Qt.PointingHandCursor)
        btn_cadastrar.clicked.connect(self.mostrar_tela_cadastro)

        form_layout = QVBoxLayout()
        form_layout.setSpacing(8)
        form_layout.setContentsMargins(0, 0, 0, 0)

        # Estilo de título dos campos
        label_style = "font-weight: bold; font-size: 14px; color: white; margin-bottom: 5px;"

        # Campo de e-mail
        email_label = QLabel("<span style='{0}'>E-mail:</span>".format(label_style))
        form_layout.addWidget(email_label)
        form_layout.addWidget(self.usuario_input)

        # Campo de senha
        senha_label = QLabel("<span style='{0}'>Senha:</span>".format(label_style))
        form_layout.addWidget(senha_label)
        form_layout.addWidget(self.senha_input)

        # Espaço antes dos botões
        form_layout.addSpacing(15)

        # Botão de login
        btn_login.setStyleSheet("background-color: #2196F3; color: white; padding: 8px; border-radius: 6px;")
        form_layout.addWidget(btn_login)

        # Botão de cadastro
        btn_cadastrar.setStyleSheet("background-color: #2196F3; color: white; padding: 8px; border-radius: 6px;")
        form_layout.addWidget(btn_cadastrar)

        card = self.criar_card(form_layout)
        self.layout.addStretch()
        self.layout.addWidget(card, alignment=Qt.AlignCenter)
        self.layout.addStretch()

    def mostrar_tela_cadastro(self):
        self.limpar_layout()

        # Inputs com placeholders
        self.nome_input = QLineEdit()
        self.nome_input.setPlaceholderText("Digite seu nome")

        self.sobrenome_input = QLineEdit()
        self.sobrenome_input.setPlaceholderText("Digite seu sobrenome")

        self.email_input = QLineEdit()
        self.email_input.setPlaceholderText("Digite seu e-mail")

        self.senha_cadastro_input = QLineEdit()
        self.senha_cadastro_input.setPlaceholderText("Digite sua senha")
        self.senha_cadastro_input.setEchoMode(QLineEdit.Password)

        # Botões
        btn_salvar = QPushButton("Salvar Cadastro")
        btn_salvar.setCursor(Qt.PointingHandCursor)
        btn_salvar.setStyleSheet("background-color: #2196F3; color: white; padding: 8px; border-radius: 6px;")
        btn_salvar.clicked.connect(self.salvar_usuario)

        btn_voltar = QPushButton("Voltar")
        btn_voltar.setCursor(Qt.PointingHandCursor)
        btn_voltar.setStyleSheet("background-color: #2196F3; color: white; padding: 8px; border-radius: 6px;")
        btn_voltar.clicked.connect(self.mostrar_tela_login)

        # Layout do formulário
        form_layout = QVBoxLayout()
        form_layout.setSpacing(10)
        form_layout.setContentsMargins(0, 0, 0, 0)

        # Estilo de label
        label_style = "font-weight: bold; font-size: 14px; color: white; margin-bottom: 5px;"

        # Adicionando os campos ao layout
        form_layout.addWidget(QLabel(f"<span style='{label_style}'>Nome:</span>"))
        form_layout.addWidget(self.nome_input)

        form_layout.addWidget(QLabel(f"<span style='{label_style}'>Sobrenome:</span>"))
        form_layout.addWidget(self.sobrenome_input)

        form_layout.addWidget(QLabel(f"<span style='{label_style}'>E-mail:</span>"))
        form_layout.addWidget(self.email_input)

        form_layout.addWidget(QLabel(f"<span style='{label_style}'>Senha:</span>"))
        form_layout.addWidget(self.senha_cadastro_input)

        form_layout.addSpacing(15)
        form_layout.addWidget(btn_salvar)
        form_layout.addWidget(btn_voltar)

        card = self.criar_card(form_layout)
        self.layout.addStretch()
        self.layout.addWidget(card, alignment=Qt.AlignCenter)
        self.layout.addStretch()

    def salvar_usuario(self):
        nome = self.nome_input.text()
        sobrenome = self.sobrenome_input.text()
        email = self.email_input.text()
        senha = self.senha_cadastro_input.text()

        if not nome or not sobrenome or not email or not senha:
            QMessageBox.warning(self, "Erro", "Preencha todos os campos!")
            return

        try:
            conexao = sqlite3.connect('usuarios.db')
            cursor = conexao.cursor()
            cursor.execute('''
                INSERT INTO usuarios (nome, sobrenome, email, senha)
                VALUES (?, ?, ?, ?)
            ''', (nome, sobrenome, email, senha))
            conexao.commit()
            conexao.close()

            QMessageBox.information(self, "Sucesso", "Usuário cadastrado com sucesso!")
            self.mostrar_tela_login()
        except sqlite3.IntegrityError:
            QMessageBox.warning(self, "Erro", "Email já cadastrado!")
        except Exception as e:
            QMessageBox.critical(self, "Erro", f"Ocorreu um erro: {e}")

    def validar_login(self):
        usuario = self.usuario_input.text()
        senha = self.senha_input.text()

        conexao = sqlite3.connect('usuarios.db')
        cursor = conexao.cursor()
        cursor.execute('''
            SELECT * FROM usuarios WHERE email = ? AND senha = ?
        ''', (usuario, senha))
        resultado = cursor.fetchone()
        conexao.close()

        if resultado:
            self.login_callback()
        else:
            QMessageBox.warning(self, "Erro de Login", "Usuário ou senha inválidos.")

    def limpar_layout(self):
        while self.layout.count():
            child = self.layout.takeAt(0)
            if child.widget():
                child.widget().deleteLater()

    def criar_card(self, form_layout):
        card = QFrame()
        card.setLayout(form_layout)
        card.setFixedWidth(340)
        card.setStyleSheet("""
            QFrame {
                background-color: #2b2b2b;
                border-radius: 12px;
                padding: 30px;
                box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.3);
            }
            QPushButton {
                background-color: #0078d7;
                color: white;
                border: none;
                padding: 8px 12px;
                border-radius: 6px;
            }
            QPushButton:hover {
                background-color: #005a9e;
            }
            QPushButton:pressed {
                background-color: #003f6d;
            }
        """)
        return card

class GraficoWidget(QWidget):
    def __init__(self):
        super().__init__()

        layout = QVBoxLayout()
        filtro_layout = QHBoxLayout()

        self.label_inicio = QLabel("Data Início:")
        self.data_inicio = QDateEdit()
        self.data_inicio.setCalendarPopup(True)
        self.data_inicio.setDate(QDate.currentDate().addMonths(-1))

        self.label_fim = QLabel("Data Fim:")
        self.data_fim = QDateEdit()
        self.data_fim.setCalendarPopup(True)
        self.data_fim.setDate(QDate.currentDate())

        self.btn_filtrar = QPushButton("Atualizar Gráficos")
        self.btn_filtrar.setCursor(Qt.PointingHandCursor)
        self.btn_filtrar.clicked.connect(self.atualizar_graficos)

        filtro_layout.addWidget(self.label_inicio)
        filtro_layout.addWidget(self.data_inicio)
        filtro_layout.addWidget(self.label_fim)
        filtro_layout.addWidget(self.data_fim)
        filtro_layout.addWidget(self.btn_filtrar)

        self.tabs = QTabWidget()

        self.tab_datas = QWidget()
        self.tab_tipos = QWidget()
        self.tab_sexo = QWidget()

        self.canvas_datas = FigureCanvas(Figure(facecolor='#1e1e1e'))
        self.canvas_tipos = FigureCanvas(Figure(facecolor='#1e1e1e'))
        self.canvas_sexo = FigureCanvas(Figure(facecolor='#1e1e1e'))

        self.tab_datas.setLayout(QVBoxLayout())
        self.tab_datas.layout().addWidget(self.canvas_datas)

        self.tab_tipos.setLayout(QVBoxLayout())
        self.tab_tipos.layout().addWidget(self.canvas_tipos)

        self.tab_sexo.setLayout(QVBoxLayout())
        self.tab_sexo.layout().addWidget(self.canvas_sexo)

        self.tabs.addTab(self.tab_datas, "Notificações por Data")
        self.tabs.addTab(self.tab_tipos, "Violência por Tipo")
        self.tabs.addTab(self.tab_sexo, "Vítimas por Sexo")

        layout.addLayout(filtro_layout)
        layout.addWidget(self.tabs)

        self.setLayout(layout)
        self.atualizar_graficos()

    def atualizar_graficos(self):
        inicio = self.data_inicio.date().toPyDate()
        fim = self.data_fim.date().toPyDate()

        client = bigquery.Client()
        query_tipos_violencia = f"""
            SELECT 'Violência física' AS tipo, COUNT(*) AS total FROM `basedosdados.br_ms_sinan.microdados_violencia`
            WHERE ocorreu_violencia_fisica = 1 AND data_notificacao BETWEEN '{inicio}' AND '{fim}'
            UNION ALL
            SELECT 'Violência psicológica', COUNT(*) FROM `basedosdados.br_ms_sinan.microdados_violencia`
            WHERE ocorreu_violencia_psicologica = 1 AND data_notificacao BETWEEN '{inicio}' AND '{fim}'
            UNION ALL
            SELECT 'Tortura', COUNT(*) FROM `basedosdados.br_ms_sinan.microdados_violencia`
            WHERE ocorreu_tortura = 1 AND data_notificacao BETWEEN '{inicio}' AND '{fim}'
            UNION ALL
            SELECT 'Violência sexual', COUNT(*) FROM `basedosdados.br_ms_sinan.microdados_violencia`
            WHERE ocorreu_violencia_sexual = 1 AND data_notificacao BETWEEN '{inicio}' AND '{fim}'
            UNION ALL
            SELECT 'Trabalho infantil', COUNT(*) FROM `basedosdados.br_ms_sinan.microdados_violencia`
            WHERE ocorreu_trabalho_infantil = 1 AND data_notificacao BETWEEN '{inicio}' AND '{fim}'
        """
        result_tipos = list(client.query(query_tipos_violencia).result())

        tipos = []
        valores = []
        for row in result_tipos:
            tipos.append(row["tipo"])
            valores.append(row["total"])

        # --- Query Datas ---
        query_datas = f"""
            SELECT data_notificacao
            FROM `basedosdados.br_ms_sinan.microdados_violencia`
            WHERE data_notificacao IS NOT NULL AND data_notificacao BETWEEN '{inicio}' AND '{fim}'
        """
        result_datas = list(client.query(query_datas).result())
        datas = []
        for row in result_datas:
            try:
                datas.append(datetime.strptime(str(row["data_notificacao"])[:10], "%Y-%m-%d"))
            except:
                pass

        # --- Query Sexo ---
        query_sexos = """
            SELECT sexo_paciente
            FROM `basedosdados.br_ms_sinan.microdados_violencia`
            WHERE sexo_paciente IS NOT NULL
            LIMIT 1000
        """
        result_sexos = list(client.query(query_sexos).result())
        sexos = [str(row["sexo_paciente"]).strip().capitalize() for row in result_sexos]

        for row in result_tipos:
            # Datas
            try:
                datas.append(datetime.strptime(str(row['data_notificacao'])[:10], '%Y-%m-%d'))
            except:
                pass

        # --- Gráfico 1: Datas ---
        self.canvas_datas.figure.clear()
        ax1 = self.canvas_datas.figure.add_subplot(111)
        contagem_datas = Counter([d.date() for d in datas])
        if contagem_datas:
            datas_ordenadas = sorted(contagem_datas.items())
            x, y = zip(*datas_ordenadas)
            ax1.plot(x, y, marker='o', color='skyblue')
            ax1.set_title("Notificações por Data", color='white')
            ax1.tick_params(colors='white')
            ax1.set_facecolor('#2b2b2b')
            self.canvas_datas.draw()

        # --- Gráfico 2: Tipos de violência ---
        self.canvas_tipos.figure.clear()
        ax2 = self.canvas_tipos.figure.add_subplot(111)

        contagem_tipos = Counter(tipos)

        labels, valores = zip(*contagem_tipos.items()) if contagem_tipos else ([], [])

        ax2.bar(labels, valores, color='orange')
        ax2.set_title("Tipos de Violência", color='white')
        ax2.tick_params(colors='white', labelrotation=45)
        ax2.set_facecolor('#2b2b2b')

        self.canvas_tipos.draw()

        # --- Gráfico 3: Sexo das vítimas ---
        self.canvas_sexo.figure.clear()
        ax3 = self.canvas_sexo.figure.add_subplot(111)
        contagem_sexos = Counter(sexos)
        if contagem_sexos:
            labels, valores = zip(*contagem_sexos.items())
            ax3.pie(valores, labels=labels, autopct='%1.1f%%', colors=plt.cm.Paired.colors, textprops={'color':'white'})
            ax3.set_title("Sexo das Vítimas", color='white')
            self.canvas_sexo.draw()

class PaginaHome(QWidget):
    def __init__(self):
        super().__init__()
        layout = QVBoxLayout()

        titulo = QLabel("V-Base: Visualização de Dados da Violência no Brasil")
        titulo.setStyleSheet("font-size: 24px; font-weight: bold; color: #00bfff")
        layout.addWidget(titulo)

        texto = QTextEdit()
        texto.setReadOnly(True)
        texto.setStyleSheet("background-color: #2b2b2b; color: #dddddd; font-size: 16px; padding: 10px; border: none")
        texto.setText("""
            <div style='text-align: justify; color: white; font-size: 14px;'>
                <p><b>V-Base — Sistema de Análise de Dados sobre Violência no Brasil</b></p>

                <p>
                    Bem-vindo à V-Base, uma plataforma analítica desenvolvida para visualização e acompanhamento de dados de notificações
                    de violência registrados no Sistema de Informação de Agravos de Notificação (SINAN), com base na base pública disponibilizada
                    pelo Ministério da Saúde e hospedada na plataforma Base dos Dados.
                </p>

                <p><b>Objetivo da Plataforma</b></p>

                <p>
                    A V-Base tem como finalidade proporcionar uma ferramenta de análise dinâmica, intuitiva e interativa para profissionais da
                    saúde pública, pesquisadores, formuladores de políticas públicas e demais interessados no fenômeno da violência no Brasil.
                    Utilizando dados reais extraídos de notificações oficiais, a plataforma permite uma leitura temporal do comportamento das
                    violências notificadas, contribuindo para diagnósticos regionais, avaliação de políticas públicas e produção científica.
                </p>

                <p><b>Sobre os Dados Utilizados</b></p>

                <p>
                    Os dados apresentados são oriundos da tabela <i>microdados_violencia</i> do dataset <i>br_ms_sinan</i>, da Base dos Dados.
                    Essa tabela contém registros anonimizados de notificações de diversos tipos de violência, incluindo:
                </p>
                <ul>
                    <li>Violência física, sexual, psicológica/moral, entre outras;</li>
                    <li>Informações sociodemográficas das vítimas e suspeitos;</li>
                    <li>Datas de notificação e ocorrência;</li>
                    <li>Local de ocorrência e circunstâncias.</li>
                </ul>

                <p><b>Funcionalidades da V-Base</b></p>

                <ul>
                    <li>Dashboard interativo com filtros de data: visualize a evolução dos casos de violência notificados em períodos personalizáveis.</li>
                    <li>Seção de gráficos especializados: acesse diferentes visualizações por tipo de violência, localidade, faixa etária, entre outros.</li>
                    <li>Segurança e controle de acesso: apenas usuários autenticados podem utilizar os recursos analíticos da plataforma.</li>
                </ul>

                <p><b>Importância do Monitoramento</b></p>

                <p>
                    Monitorar dados de violência é essencial para:
                </p>
                <ul>
                    <li>Detectar padrões e tendências;</li>
                    <li>Planejar intervenções eficazes;</li>
                    <li>Avaliar resultados de políticas públicas;</li>
                    <li>Promover ações de prevenção e apoio às vítimas.</li>
                </ul>
            </div>
        """)

        layout.addWidget(texto)
        self.setLayout(layout)

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("V-Base - Análise de Dados da Violência")
        self.resize(1100, 700)

        self.stack = QStackedLayout()
        self.main_interface = QWidget()
        self.stack.addWidget(AuthWidget(self.carregar_interface_principal))
        self.stack.addWidget(self.main_interface)

        container = QWidget()
        container.setLayout(self.stack)
        self.setCentralWidget(container)

    def carregar_interface_principal(self):
        try:
            menu = QVBoxLayout()
            self.btn_home = QPushButton("🏠 Início")
            self.btn_g1 = QPushButton("📈 Gráfico 1")
            self.btn_g2 = QPushButton("📊 Gráfico 2")
            self.btn_g3 = QPushButton("📋 Gráfico 3")

            for btn in [self.btn_home, self.btn_g1, self.btn_g2, self.btn_g3]:
                btn.setCursor(Qt.PointingHandCursor)
                btn.setStyleSheet("padding: 10px; font-size: 14px;")
                menu.addWidget(btn)
            menu.addStretch()

            self.paginas = QStackedLayout()
            self.home = PaginaHome()
            self.grafico1 = GraficoWidget()
            self.grafico2 = GraficoWidget()
            self.grafico3 = GraficoWidget()

            self.paginas.addWidget(self.home)
            self.paginas.addWidget(self.grafico1)
            self.paginas.addWidget(self.grafico2)
            self.paginas.addWidget(self.grafico3)

            self.btn_home.clicked.connect(lambda: self.paginas.setCurrentIndex(0))
            self.btn_g1.clicked.connect(lambda: self.paginas.setCurrentIndex(1))
            self.btn_g2.clicked.connect(lambda: self.paginas.setCurrentIndex(2))
            self.btn_g3.clicked.connect(lambda: self.paginas.setCurrentIndex(3))

            hbox = QHBoxLayout()
            hbox.addLayout(menu, 1)
            hbox.addLayout(self.paginas, 5)

            self.main_interface.setLayout(hbox)
            self.stack.setCurrentIndex(1)
        except Exception as e:
            QMessageBox.critical(self, "Erro crítico", str(e))



if __name__ == '__main__':
    app = QApplication(sys.argv)
    app.setStyle("Fusion")

    dark_palette = QPalette()
    dark_palette.setColor(QPalette.Window, QColor(30, 30, 30))
    dark_palette.setColor(QPalette.WindowText, QColor(220, 220, 220))
    dark_palette.setColor(QPalette.Base, QColor(45, 45, 45))
    dark_palette.setColor(QPalette.AlternateBase, QColor(30, 30, 30))
    dark_palette.setColor(QPalette.ToolTipBase, QColor(255, 255, 255))
    dark_palette.setColor(QPalette.ToolTipText, QColor(0, 0, 0))
    dark_palette.setColor(QPalette.Text, QColor(220, 220, 220))
    dark_palette.setColor(QPalette.Button, QColor(45, 45, 45))
    dark_palette.setColor(QPalette.ButtonText, QColor(220, 220, 220))
    dark_palette.setColor(QPalette.BrightText, QColor(255, 0, 0))
    dark_palette.setColor(QPalette.Highlight, QColor(0, 120, 215))
    dark_palette.setColor(QPalette.HighlightedText, QColor(0, 0, 0))
    app.setPalette(dark_palette)

    window = MainWindow()
    window.show()
    sys.exit(app.exec_())
