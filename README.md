---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.14.4
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

# Sistemas Inteligentes 2021/2022

## Mini-projeto 2: Quadrados Latinos

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/10_x10_lateinisches_quadrat.svg/640px-10_x10_lateinisches_quadrat.svg.png" width="80px"> </img>

<!-- #region -->
### Introdução

A imagem acima [(link)](https://wikimedia.org/api/rest_v1/media/math/render/svg/d9d28d25e16d5faea934a2ca12e23c1f57a31e39) é um exemplo de um [quadrado latino](https://pt.wikipedia.org/wiki/Quadrado_latino). Pode ser representado por uma matriz *n x n* em que cada linha e coluna contêm exatamente uma vez um símbolo ou número de entre *n* símbolos possíveis.

Uma variação do Quadrado Latino é o puzzle de [Futoshiki](https://pt.wikipedia.org/wiki/Futoshiki). Este puzzle é representado exatamente da mesma forma, por uma matriz *n x n*, em que os números de 1 a *n* só podem aparecer uma única vez na mesma linha ou coluna da matriz. Para além disso, têm ainda de satisfazer as desigualdades colocadas em algumas células adjacentes. Este [link](https://www.futoshiki.org/) pode ser usado para gerar puzzles e testar a implementação.

### Objetivos

O objetivo deste projeto é formular esta classe de puzzles como um CSP.
Este trabalho está dividido em dois sub-problemas, o dos quadrados latinos normal, e a sua variação Futoshiki. O objetivo é formular cada problema de acordo com um CSP, usando a implementação disponibilizada pelo módulo `csp.py` que tem sido utilizado nas aulas PL, e resolver os problemas utilizando os algoritmos de procura adequados. Deverá apresentar a sua implementação bem como os testes que considerar necessários para demonstrar que a formulação está bem feita.




