# Story — Semeadora SSM 23 (reformada) · Sempre Nova

Formato 1080×1920 (9:16), padrão de story de máquina da Sempre Nova.

Estrutura (mesma do story do SSM 27, na variante "reformada"):
- foto full-bleed com scrim de céu (topo) e scrim de chão (base)
- filete vermelho + kicker `SEMEADORA DE PLANTIO DIRETO`
- modelo em caixa alta condensada (`SSM 23`)
- status em duas linhas (`UNIDADE REFORMADA / REVISADA E PRONTA PARA PLANTAR`)
- filete verde + três colunas de dados técnicos
- faixa inferior sólida com a logo Sempre Nova

Arquivos:
- `story.html` — arte editável (todo o texto e as medidas ficam aqui)
- `bg.jpg` — foto de fundo (trocar por uma foto da própria SSM 23)
- `story_ssm23.png` — render final

Para gerar o PNG:

    chromium --headless --window-size=1080,1920 \
      --screenshot=story_ssm23.png story.html
