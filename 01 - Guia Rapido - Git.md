# Guia Rápido - Git
## O que é?
Git é uma ferramenta de versionamento de software, existem diversos sites que fazem o sistema de Git, o mais conhecido deles é o GitHub (http://www.github.com).
- - -
## Instalação
Em distribuições Debian-like `sudo apt-get install git`, RedHat-like `sudo yum install git`. 
- - -
## Criar/Clonar projetos
- `git init` cria um repositório locar na máquina.

- `git clone [url de clone]` clona um repositório.

- - -
## Snapshots
- `git add [nome do arquivo/pasta]` adiciona arquivo a monitoramento.
- `git add .` adiciona todos os arquivos do diretório a monitoramento.
- `git status` retorna o estado atual do git detalhado.
 - `git status -s` retorna o resumo de mudança dos arquivos.
- `git commit -a` comita todas as mudanças do projeto.
 -  `git commit -a -m [mensagem]` comita as mudanças com mensagem.

- - -
## Referencias
- [GitRef]
- [Git-Scm]

[GitRef]:http://gitref.org/
[Git-Scm]:http://git-scm.com/book/pt-br
