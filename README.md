# Treino CTS-2 VR

Treinamento em realidade virtual (celular, Android e iPhone) da manutenção preventiva da máquina de chave CTS-2, conforme o PRO-001122, Anexo 5.

- App: abra `index.html` pelo GitHub Pages.
- Tela do avaliador: `avaliador.html`.

O site é estático (three.js r128, PeerJS). A branch `gh-pages` é a que vai ao ar.

## Treinamentos a partir dos PROs

Cada procedimento vira um arquivo JSON em `pros/` (lista em `pros/indice.json`). O app monta sozinho o cartão da biblioteca, o menu, a escolha de papel (coordenador ou executante), as etapas, as falas de rádio e os desafios de cada PRO.

- Guia para converter um PRO novo: `pros/COMO-CRIAR-UM-PRO.md`.
- Para testar um PRO novo sem publicar: biblioteca → "Carregar PRO (.json)" (fica só no aparelho).
- Rádio: falas entre a equipe de campo e o CCP têm bip e "câmbio"; instruções ao participante e leitura do quadro não têm.
