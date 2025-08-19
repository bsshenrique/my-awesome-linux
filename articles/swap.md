# Swap
## Swap space
*Swap space* destina-se a armazenar páginas de memória virtual ou permitir a suspensão para disco.

Quando utilizado para armazenar páginas de memória virtual, ele funciona como uma extensão da memória física (RAM), realizando *swapping* quando não há mais RAM disponível.

*Swapping* é o processo em que o sistema operacional move temporariamente páginas de memória virtual inativas ou usadas com menos frequência da RAM para o local indicado como *swap*.
 
> [!TIP]
> Existem diversas maneiras de uso de um *swap space*.
>
> Em *swap partition*, uma partição dedicada ao *swap*.  
> Usando um *swap file*, que é um arquivo em disco destinado a *swap*.  
> Em `zram`, `zswap` e diversas outras maneiras.
>
> Usar *swap* é algo opcional, maneira escolhida depende do hardware, de como o dispositivo é usado, e contará com vantagens e desvantagens.  
> Veja maiores detalhes em [swap](https://wiki.archlinux.org/title/Swap).

> [!IMPORTANT]
> Não use *swap* apenas por usar, valide se ele é necessário para você.
>
> Por exemplo, *swap* em `zram` ocupa parte da RAM e pode elevar o consumo de CPU devido à compressão e gerar impacto em processadores menos potentes.  
> Já uma partição *swap* pode nunca ser utilizada em sistemas com grande quantidade de RAM.
>
> No meu caso, em que tenho uma quantidade significativa de RAM, não uso suspensão, utilizo poucos programas simultaneamente e não quero alocar blocos comprimidos em RAM, não faz muito sentido o uso de *swap*.  
> Porém, visando evitar qualquer problema, gosto de utilizar um `swap file` com uma capacidade baixa, algo como 4G.  
> É um espaço que não fará falta e provavelmente nunca será usado, mas estará disponível caso necessário.


## zram
`zram` é um módulo do *kernel* Linux utilizado para criar um dispositivo de bloco compactado em RAM.  
Um de seus usos é utilizar blocos criados em RAM como *swap*.


## zswap
Recurso do *kernel* Linux que fornece um *cache* RAM compactado para o *swap*.  
Possibilita que páginas sejam compactadas e armazenadas em um *pool* na RAM.  
Quando não há mais espaço no *pool* ou na própria RAM, a página é descompactada e transferida para o *swap*.


## Swappiness
O *kernel* Linux permite configurar a tendência de uso do swap através do parâmetro `swappiness`.  
O valor pode variar entre 0 e 100, e indicará a tendência de usar swap, exemplo:

```plaintext
0   Não deverá usar swap
1   Quando o uso de RAM >= 99%
10  Quando o uso de RAM >= 90%
20  Quando o uso de RAM >= 80%
30  Quando o uso de RAM >= 70%
60  Quando o uso de RAM >= 40% (padrão)
```

Para definir de forma permanente o valor do `swappiness` cria um arquivo de configuração `sysctl.d`:

```bash
echo "vm.swappiness = 25" | sudo tee /etc/sysctl.d/99-swappiness.conf

# Aplicar imediatamente
sudo sysctl --system 

# Visualizar o valor
sysctl vm.swappiness
# ou
cat /proc/sys/vm/swappiness
```
