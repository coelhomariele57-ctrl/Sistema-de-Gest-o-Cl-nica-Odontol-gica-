# Sistema-de-Gest-o-Cl-nica-Odontol-gica-
agendamento de pacientes, controle de estoque e dados 
# ============================================================
#   SISTEMA DE GESTÃO - CONSULTÓRIO ODONTOLÓGICO
#   Conceitos aplicados: Algoritmos, Estruturas de Dados,
#   Orientação a Objetos, Banco de Dados (simulado),
#   Lógica Matemática e Estatística
# ============================================================

from datetime import date, datetime
import statistics


# ============================================================
# 1. ORIENTAÇÃO A OBJETOS — Classes do sistema
# ============================================================

class Pessoa:
    """Classe base (herança) para Paciente e Dentista."""

    def __init__(self, nome, cpf, telefone):
        self.nome = nome
        self.cpf = cpf
        self.telefone = telefone

    def apresentar(self):
        return f"Nome: {self.nome} | CPF: {self.cpf} | Tel: {self.telefone}"


class Paciente(Pessoa):
    """Herda de Pessoa. Representa um paciente da clínica."""

    def __init__(self, nome, cpf, telefone, data_nascimento):
        super().__init__(nome, cpf, telefone)
        self.data_nascimento = data_nascimento  # formato: "DD/MM/AAAA"
        self.historico = []       # lista de consultas realizadas
        self.retorno_pendente = False

    def calcular_idade(self):
        dia, mes, ano = map(int, self.data_nascimento.split("/"))
        hoje = date.today()
        idade = hoje.year - ano - ((hoje.month, hoje.day) < (mes, dia))
        return idade

    def adicionar_consulta(self, consulta):
        self.historico.append(consulta)

    def apresentar(self):
        base = super().apresentar()
        return f"{base} | Idade: {self.calcular_idade()} anos"


class Dentista(Pessoa):
    """Herda de Pessoa. Representa um dentista da clínica."""

    def __init__(self, nome, cpf, telefone, cro, especialidade):
        super().__init__(nome, cpf, telefone)
        self.cro = cro
        self.especialidade = especialidade

    def apresentar(self):
        base = super().apresentar()
        return f"{base} | CRO: {self.cro} | Especialidade: {self.especialidade}"


class Consulta:
    """Representa uma consulta agendada ou realizada."""

    def __init__(self, paciente, dentista, data, horario, procedimento, valor):
        self.paciente = paciente
        self.dentista = dentista
        self.data = data          # formato: "DD/MM/AAAA"
        self.horario = horario    # formato: "HH:MM"
        self.procedimento = procedimento
        self.valor = valor
        self.realizada = False
        self.confirmada = False

    def confirmar(self):
        self.confirmada = True
        print(f"  ✔ Consulta de {self.paciente.nome} confirmada para {self.data} às {self.horario}.")

    def realizar(self):
        self.realizada = True
        self.paciente.adicionar_consulta(self)
        print(f"  ✔ Consulta realizada: {self.procedimento} — R$ {self.valor:.2f}")

    def __str__(self):
        status = "Realizada" if self.realizada else ("Confirmada" if self.confirmada else "Pendente")
        return (f"  [{status}] {self.data} {self.horario} | "
                f"Paciente: {self.paciente.nome} | "
                f"Procedimento: {self.procedimento} | "
                f"Valor: R$ {self.valor:.2f}")


# ============================================================
# 2. ESTRUTURAS DE DADOS — Fila de espera e Estoque
# ============================================================

class FilaEspera:
    """
    Fila de espera usando lista como fila (FIFO).
    FIFO = First In, First Out (primeiro a entrar, primeiro a sair).
    """

    def __init__(self):
        self._fila = []

    def entrar(self, paciente):
        self._fila.append(paciente)
        print(f"  → {paciente.nome} entrou na fila. Posição: {len(self._fila)}")

    def chamar_proximo(self):
        if self._fila:
            proximo = self._fila.pop(0)
            print(f"  → Chamando: {proximo.nome}")
            return proximo
        else:
            print("  → Fila vazia.")
            return None

    def tamanho(self):
        return len(self._fila)

    def mostrar(self):
        if not self._fila:
            print("  Fila vazia.")
        else:
            for i, p in enumerate(self._fila, 1):
                print(f"  {i}. {p.nome}")


class EstoqueInsumos:
    """
    Estoque de materiais usando pilha (LIFO) para controle de validade.
    LIFO = Last In, First Out (último a entrar, primeiro a sair).
    Simula: materiais mais recentes são usados primeiro
    para evitar vencimento.
    """

    def __init__(self):
        self._pilha = []   # cada item: {"material": ..., "quantidade": ..., "validade": ...}

    def adicionar(self, material, quantidade, validade):
        item = {"material": material, "quantidade": quantidade, "validade": validade}
        self._pilha.append(item)
        print(f"  + Adicionado: {material} ({quantidade} un.) | Validade: {validade}")

    def usar(self, material, quantidade_usar):
        # Busca o item pelo nome e desconta a quantidade
        for item in reversed(self._pilha):
            if item["material"].lower() == material.lower():
                if item["quantidade"] >= quantidade_usar:
                    item["quantidade"] -= quantidade_usar
                    print(f"  - Usado: {quantidade_usar} un. de {material}. "
                          f"Restante: {item['quantidade']} un.")
                    return
                else:
                    print(f"  ✗ Estoque insuficiente de {material}. "
                          f"Disponível: {item['quantidade']} un.")
                    return
        print(f"  ✗ Material '{material}' não encontrado no estoque.")

    def verificar_vencimentos(self):
        print("\n  === Verificação de Validade ===")
        hoje = date.today()
        alerta = False
        for item in self._pilha:
            dia, mes, ano = map(int, item["validade"].split("/"))
            validade = date(ano, mes, dia)
            dias_restantes = (validade - hoje).days
            # LÓGICA: alerta se vence em menos de 30 dias E ainda tem estoque
            if dias_restantes <= 30 and item["quantidade"] > 0:
                print(f"  ⚠ ALERTA: {item['material']} vence em {dias_restantes} dias!")
                alerta = True
        if not alerta:
            print("  ✔ Nenhum material próximo do vencimento.")

    def mostrar(self):
        if not self._pilha:
            print("  Estoque vazio.")
        else:
            for item in self._pilha:
                print(f"  • {item['material']}: {item['quantidade']} un. "
                      f"| Validade: {item['validade']}")


# ============================================================
# 3. BANCO DE DADOS SIMULADO — Repositório central
# ============================================================

class BancoDados:
    """
    Simula um banco de dados usando dicionários e listas.
    Em sistemas reais: usaria SQL (SQLite, MySQL, PostgreSQL).
    """

    def __init__(self):
        self._pacientes = {}    # chave: CPF
        self._dentistas = {}    # chave: CRO
        self._consultas = []    # lista de objetos Consulta

    # --- Pacientes ---
    def cadastrar_paciente(self, paciente):
        if paciente.cpf in self._pacientes:
            print(f"  ✗ CPF {paciente.cpf} já cadastrado.")
        else:
            self._pacientes[paciente.cpf] = paciente
            print(f"  ✔ Paciente '{paciente.nome}' cadastrado.")

    def buscar_paciente(self, cpf):
        return self._pacientes.get(cpf, None)

    def listar_pacientes(self):
        if not self._pacientes:
            print("  Nenhum paciente cadastrado.")
        else:
            for p in self._pacientes.values():
                print(f"  • {p.apresentar()}")

    # --- Dentistas ---
    def cadastrar_dentista(self, dentista):
        self._dentistas[dentista.cro] = dentista
        print(f"  ✔ Dentista '{dentista.nome}' cadastrado.")

    def buscar_dentista(self, cro):
        return self._dentistas.get(cro, None)

    # --- Consultas ---
    def agendar_consulta(self, consulta):
        # ALGORITMO: verifica conflito de horário antes de agendar
        for c in self._consultas:
            if (c.dentista.cro == consulta.dentista.cro and
                    c.data == consulta.data and
                    c.horario == consulta.horario and
                    not c.realizada):
                print(f"  ✗ Conflito! {consulta.dentista.nome} já tem consulta "
                      f"em {consulta.data} às {consulta.horario}.")
                return False
        self._consultas.append(consulta)
        print(f"  ✔ Consulta agendada: {consulta.paciente.nome} "
              f"em {consulta.data} às {consulta.horario}.")
        return True

    def consultas_do_dia(self, data):
        return [c for c in self._consultas if c.data == data]

    def todas_consultas(self):
        return self._consultas


# ============================================================
# 4. ESTATÍSTICA — Relatório gerencial
# ============================================================

def gerar_relatorio(banco):
    """
    Gera relatório com indicadores estatísticos do consultório.
    Usa o módulo 'statistics' do Python.
    """
    print("\n" + "=" * 55)
    print("         RELATÓRIO GERENCIAL DO CONSULTÓRIO")
    print("=" * 55)

    consultas = banco.todas_consultas()
    realizadas = [c for c in consultas if c.realizada]

    # Total de consultas
    print(f"\n  Total de consultas agendadas : {len(consultas)}")
    print(f"  Consultas realizadas         : {len(realizadas)}")
    print(f"  Consultas pendentes          : {len(consultas) - len(realizadas)}")

    if not realizadas:
        print("\n  Sem dados financeiros ainda.")
        return

    # LÓGICA MATEMÁTICA: taxa de absenteísmo
    # (consultas agendadas E não realizadas) / total * 100
    nao_realizadas = len(consultas) - len(realizadas)
    taxa_absenteismo = (nao_realizadas / len(consultas)) * 100 if consultas else 0
    print(f"\n  Taxa de absenteísmo          : {taxa_absenteismo:.1f}%")

    # Estatística dos valores
    valores = [c.valor for c in realizadas]
    print(f"\n  --- Indicadores Financeiros ---")
    print(f"  Receita total                : R$ {sum(valores):.2f}")
    print(f"  Ticket médio por consulta    : R$ {statistics.mean(valores):.2f}")
    print(f"  Valor mediano                : R$ {statistics.median(valores):.2f}")
    if len(valores) > 1:
        print(f"  Desvio padrão dos valores    : R$ {statistics.stdev(valores):.2f}")

    # Procedimentos mais realizados
    print(f"\n  --- Procedimentos Realizados ---")
    contagem = {}
    for c in realizadas:
        contagem[c.procedimento] = contagem.get(c.procedimento, 0) + 1
    # Ordenação: algoritmo de ordenação por valor decrescente
    ranking = sorted(contagem.items(), key=lambda x: x[1], reverse=True)
    for procedimento, qtd in ranking:
        print(f"  • {procedimento}: {qtd} vez(es)")

    # Faixa etária dos pacientes atendidos
    print(f"\n  --- Faixa Etária dos Pacientes ---")
    idades = [c.paciente.calcular_idade() for c in realizadas]
    if idades:
        print(f"  Idade mínima : {min(idades)} anos")
        print(f"  Idade máxima : {max(idades)} anos")
        print(f"  Idade média  : {statistics.mean(idades):.1f} anos")

    print("\n" + "=" * 55)


# ============================================================
# 5. ALGORITMO — Lembrete automático por lógica condicional
# ============================================================

def verificar_lembretes(banco, data_amanha):
    """
    Algoritmo de verificação de lembretes.
    Regra lógica:
      SE consulta agendada para amanhã
      E não confirmada
      E paciente com retorno pendente
      ENTÃO emitir alerta
    """
    print(f"\n  === Lembretes para {data_amanha} ===")
    consultas = banco.consultas_do_dia(data_amanha)
    alertas = 0
    for c in consultas:
        # Operadores lógicos AND (and) aplicados
        if not c.confirmada and not c.realizada:
            print(f"  📱 Enviar lembrete para {c.paciente.nome} "
                  f"({c.paciente.telefone}) — {c.horario} | {c.procedimento}")
            alertas += 1
    if alertas == 0:
        print("  ✔ Todas as consultas já confirmadas.")


# ============================================================
# DEMONSTRAÇÃO DO SISTEMA
# ============================================================

def main():
    print("=" * 55)
    print("   SISTEMA DE GESTÃO — CONSULTÓRIO ODONTOLÓGICO")
    print("=" * 55)

    # --- Instanciando o banco de dados ---
    db = BancoDados()

    # --- Cadastro de dentistas ---
    print("\n[1] CADASTRO DE DENTISTAS")
    print("-" * 40)
    dra_ana = Dentista("Dra. Ana Paula", "111.222.333-44", "(32) 99999-1111",
                       "CRO-MG 12345", "Clínica Geral")
    dr_carlos = Dentista("Dr. Carlos Henrique", "555.666.777-88", "(32) 99999-2222",
                         "CRO-MG 67890", "Ortodontia")
    db.cadastrar_dentista(dra_ana)
    db.cadastrar_dentista(dr_carlos)

    # --- Cadastro de pacientes ---
    print("\n[2] CADASTRO DE PACIENTES")
    print("-" * 40)
    p1 = Paciente("Maria Silva", "123.456.789-00", "(32) 98888-0001", "15/03/1990")
    p2 = Paciente("João Oliveira", "987.654.321-00", "(32) 98888-0002", "22/07/1985")
    p3 = Paciente("Fernanda Costa", "456.789.123-00", "(32) 98888-0003", "10/11/2000")
    p4 = Paciente("Lucas Mendes", "321.654.987-00", "(32) 98888-0004", "05/01/1975")
    db.cadastrar_paciente(p1)
    db.cadastrar_paciente(p2)
    db.cadastrar_paciente(p3)
    db.cadastrar_paciente(p4)

    print("\n  Pacientes cadastrados:")
    db.listar_pacientes()

    # --- Agendamento de consultas ---
    print("\n[3] AGENDAMENTO DE CONSULTAS")
    print("-" * 40)
    c1 = Consulta(p1, dra_ana, "10/06/2025", "09:00", "Limpeza", 150.00)
    c2 = Consulta(p2, dra_ana, "10/06/2025", "10:00", "Restauração", 280.00)
    c3 = Consulta(p3, dr_carlos, "10/06/2025", "09:00", "Consulta Ortodôntica", 200.00)
    c4 = Consulta(p4, dra_ana, "10/06/2025", "09:00", "Extração", 350.00)  # conflito!
    c5 = Consulta(p1, dra_ana, "11/06/2025", "14:00", "Retorno", 100.00)

    db.agendar_consulta(c1)
    db.agendar_consulta(c2)
    db.agendar_consulta(c3)
    db.agendar_consulta(c4)  # deve detectar conflito
    db.agendar_consulta(c5)

    # --- Confirmações ---
    print("\n[4] CONFIRMAÇÃO DE CONSULTAS")
    print("-" * 40)
    c1.confirmar()
    c2.confirmar()
    c3.confirmar()
    # c5 NÃO confirmada → vai aparecer no lembrete

    # --- Lembretes automáticos ---
    print("\n[5] VERIFICAÇÃO DE LEMBRETES")
    print("-" * 40)
    verificar_lembretes(db, "11/06/2025")

    # --- Realização das consultas ---
    print("\n[6] REALIZANDO CONSULTAS DO DIA")
    print("-" * 40)
    print("  Consultas do dia 10/06/2025:")
    for c in db.consultas_do_dia("10/06/2025"):
        print(c)

    print("\n  Realizando consultas confirmadas...")
    c1.realizar()
    c2.realizar()
    c3.realizar()

    # --- Fila de espera ---
    print("\n[7] FILA DE ESPERA")
    print("-" * 40)
    fila = FilaEspera()
    fila.entrar(p4)
    fila.entrar(p1)
    fila.entrar(p2)
    print("\n  Fila atual:")
    fila.mostrar()
    print("\n  Chamando próximo paciente:")
    fila.chamar_proximo()
    print(f"\n  Pacientes restantes na fila: {fila.tamanho()}")

    # --- Controle de estoque ---
    print("\n[8] CONTROLE DE ESTOQUE (PILHA)")
    print("-" * 40)
    estoque = EstoqueInsumos()
    estoque.adicionar("Luvas Nitrílicas", 100, "31/12/2025")
    estoque.adicionar("Anestésico Lidocaína", 50, "15/06/2025")  # próximo do vencimento
    estoque.adicionar("Resina Composta", 30, "30/09/2025")
    estoque.adicionar("Fio Dental", 200, "01/01/2026")

    print("\n  Usando materiais:")
    estoque.usar("Luvas Nitrílicas", 10)
    estoque.usar("Anestésico Lidocaína", 3)
    estoque.usar("Cimento", 5)  # material inexistente

    print("\n  Estoque atual:")
    estoque.mostrar()
    estoque.verificar_vencimentos()

    # --- Busca de paciente por CPF ---
    print("\n[9] BUSCA POR CPF (algoritmo de busca em dicionário)")
    print("-" * 40)
    cpf_busca = "987.654.321-00"
    resultado = db.buscar_paciente(cpf_busca)
    if resultado:
        print(f"  ✔ Paciente encontrado: {resultado.apresentar()}")
        print(f"  Histórico de consultas: {len(resultado.historico)} consulta(s)")
    else:
        print(f"  ✗ CPF {cpf_busca} não encontrado.")

    # --- Relatório estatístico ---
    print("\n[10] RELATÓRIO GERENCIAL")
    gerar_relatorio(db)


if __name__ == "__main__":
    main()
