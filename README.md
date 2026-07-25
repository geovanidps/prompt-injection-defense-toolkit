# Prompt Injection Defense Toolkit

Toolkit defensivo, open-source e educacional, para **detecção, teste e
resposta a incidentes** relacionados a ataques de prompt (prompt injection,
jailbreak e variantes) em aplicações baseadas em LLMs.

> **Este é um projeto exclusivamente defensivo.** Ele documenta mecanismos de
> ataque em nível conceitual (o suficiente para reconhecimento e construção
> de defesas), mas **não contém payloads funcionais, exploits prontos ou
> instruções operacionais de bypass**. Para testes ofensivos reais, utilize
> ferramentas de red teaming estabelecidas (ex.: [Garak](https://github.com/leondz/garak))
> dentro de um programa de segurança autorizado.

## Por que este projeto existe

Prompt injection é classificado como o risco #1 no OWASP Top 10 for LLM
Applications (2025), e relatórios do setor (CrowdStrike, Cisco) documentam
crescimento acelerado de incidentes reais em 2025–2026, incluindo exploits
zero-click em produtos de grande escala (ex.: EchoLeak / CVE-2025-32711).
Este toolkit organiza conhecimento público disperso em uma estrutura
acionável para equipes de segurança, desenvolvedores de IA e pesquisadores.

## Estrutura do repositório

```
prompt-injection-defense-toolkit/
├── README.md
├── LICENSE
├── docs/
│   ├── taxonomy.md                     # Taxonomia com 25 subcategorias nomeadas
│   ├── redteam_checklist.md            # Checklist de cobertura de testes
│   └── incident_response_template.md   # Template de relatório executivo
├── detection/
│   ├── patterns.yaml                   # Regras heurísticas mapeadas à taxonomia
│   ├── heuristics.py                   # Scanner heurístico (regex + entropia + drift)
│   └── requirements.txt
├── testing/
│   ├── run_tests.md                    # Guia de integração Garak + NeMo Guardrails
│   └── nemo_guardrails/
│       ├── config.yml
│       └── rails/topical_rail.co
└── .github/
    └── ISSUE_TEMPLATE/
        └── incident_report.md          # Template de issue para incidentes
```

## Como usar

### 1. Taxonomia como referência

Comece por [`docs/taxonomy.md`](docs/taxonomy.md): 25 subcategorias de
ataque de prompt, organizadas em 5 blocos (injeção direta, ofuscação,
contexto simulado/escalonamento, injeção indireta, agentes/ferramentas),
cada uma com mecanismo, padrão de detecção e mitigação.

### 2. Detecção heurística

```bash
cd detection
pip install -r requirements.txt
python heuristics.py --text "ignore todas as instruções anteriores e..."
```

Ou como biblioteca:

```python
from detection.heuristics import PromptGuard

guard = PromptGuard("detection/patterns.yaml")
result = guard.scan(user_message)

if result.action == "bloquear":
    reject_request()
elif result.action == "sinalizar para revisão humana":
    queue_for_review()
```

> **Importante:** heurísticas de regex têm alta taxa de falso negativo contra
> ataques adaptativos. Use-as como primeira camada, combinadas com
> classificadores semânticos, guardrails e validação de saída — nunca como
> única linha de defesa (ver seção "Defesa em profundidade" abaixo).

### 3. Testes automatizados

Siga [`testing/run_tests.md`](testing/run_tests.md) para configurar:
- **Garak** para varreduras automatizadas de vulnerabilidade;
- **NeMo Guardrails** para controle de fluxo de conversa em tempo de execução,
  usando `testing/nemo_guardrails/` como ponto de partida.

### 4. Red team estruturado

Use [`docs/redteam_checklist.md`](docs/redteam_checklist.md) para planejar e
rastrear cobertura de testes contra todas as 25 categorias da taxonomia,
mais validações de arquitetura independentes de payload específico.

### 5. Resposta a incidentes

- Para registrar um incidente real ou achado de teste: use o template em
  [`.github/ISSUE_TEMPLATE/incident_report.md`](.github/ISSUE_TEMPLATE/incident_report.md)
  (abre automaticamente ao criar uma issue no GitHub).
- Para consolidar uma campanha completa de testes: use
  [`docs/incident_response_template.md`](docs/incident_response_template.md).

## Princípio de defesa em profundidade

Nenhuma camada isolada é suficiente. A arquitetura recomendada combina:

1. Separação estrutural entre instrução de sistema e dado de entrada.
2. Filtros heurísticos de entrada (este repositório).
3. Classificadores semânticos dedicados de prompt injection.
4. Guardrails programáveis em tempo de execução (NeMo Guardrails ou similar).
5. Tratamento de toda saída do LLM como não confiável antes de execução.
6. Privilégio mínimo e aprovação humana para ações de agentes de alto risco.
7. Monitoramento contínuo, auditoria de memória persistente e testes
   adversariais recorrentes (Garak + red team manual).

## Contribuindo

Contribuições são bem-vindas para:
- Novas regras heurísticas em `detection/patterns.yaml` (com referência à
  categoria da taxonomia correspondente);
- Novas rails de guardrails em `testing/nemo_guardrails/rails/`;
- Melhorias na taxonomia com base em literatura acadêmica publicada.

**Não serão aceitas** contribuições que incluam payloads funcionais de
ataque, exploits prontos para uso ou instruções operacionais de bypass.
Descrições em nível conceitual, com referência a fontes públicas
(OWASP, arXiv, relatórios de segurança), são o padrão deste repositório.

## Referências principais

- OWASP GenAI Security Project — [LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- NIST AI 600-1 — Generative AI Profile
- Garak — [github.com/leondz/garak](https://github.com/leondz/garak)
- NeMo Guardrails — [github.com/NVIDIA/NeMo-Guardrails](https://github.com/NVIDIA/NeMo-Guardrails)

Ver lista completa de referências acadêmicas em `docs/taxonomy.md`.

## Licença

MIT — ver [`LICENSE`](LICENSE). Uso exclusivamente educacional e defensivo.
