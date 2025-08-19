# Shell
Além do `sh` (Bourne Shell) e `bash` (Bourne Again SHell), outros *shells* muito populares são o `zsh` (Z Shell) e `fish` (Friendly Interactive Shell).

## Zsh
O `zsh` oferece uma enorme capacidade de customização e diversos *plugins*.  
É o *shell* que eu mais tenho familiaridade.

Instale o `zsh` e em seguida abra o assistente de configuração:  
`sudo pacman -S zsh`  
`zsh`

### Oh My Zsh
Instale o [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh).

> [!TIP]
> Ao usar o `Oh My Zsh` sempre realize a instalação de temas e *plugins* por meio dele.
> 
> É possível criar um script para atualizar os temas e *plugins* instalados.  
> Basta um `git pull` nos diretórios em `~/.oh-my-zsh/custom/plugins/` e `~/.oh-my-zsh/custom/themes/`.

Instale o tema [Powerlevel10k](https://github.com/romkatv/powerlevel10k).

Instale os *plugins*:  
[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions);  
[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting).

> [!TIP]
> O comportamento padrão de sugestão após colar algo é bastante irritante e ele pode ser removido conforme o exemplo abaixo.

```bash
plugins=(...)

source $ZSH/oh-my-zsh.sh

# User configuration
ZSH_AUTOSUGGEST_CLEAR_WIDGETS+=(bracketed-paste)
```
