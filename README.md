# Como fazer o módulo Arquivos poliglotas/PDF Bootloader da Universidade Hacker no Linux

Esse repositório tem como objetivo ajudar outros alunos da Universidade Hacker que, assim como eu, possuem como sistema operacional o GNU/Linux e não desejam ou não podem utilizar o Windows nativamente. 

De antemão deixo avisado que vai ser uma experiência **um pouco** frustrante, no sentido de que algumas coisas não vão funcionar como deveriam e serão mais difíceis do que mostradas no curso. Entretanto, você pode usar essa experiência para aprender mais sobre Linux, aprender a se virar quando o ambiente não for o ideal e improvisar quando ninguém tem uma resposta pronta do que fazer!

Durante o módulo, eu utilizei o Zorin OS 17.3, o que significa que estou utilizando uma distro baseada em Ubuntu, logo, **quase** tudo que eu mostrar aqui deve funcionar igual ou ao menos bastante similar para Ubuntu e outras distros baseadas em Ubuntu. Você também não **deve** ter problemas com Debian e distro baseadas em Debian.

Espero que esse material te ajude, qualquer correção ou sugestão pode falar comigo no twitter pelo user @perlutech

Existe um pré-primeiro passso que é: atualize os pacotes e a lista de pacotes no seu sistem com ```sudo apt update && sudo apt upgrade```

## Primeiro passo: instalando ferramentas.

Esse passo é autoexplicativo, você vai precisar de algumas ferramentas diferentes das apresentadas no curso e aqui listarei as que melhor me serviram. Você não vai entender porque vai precisar de tudo isso nas primeiras aulas, mas conforme for avançando vai ver que tudo tem uma utilidade.

### Wine

O Wine é um software que vai te ajudar a rodar executáveis do Windows no Linux. Simples assim. Você pode encontrar mais sobre o [Wine](https://markdownlivepreview.com/) aqui.
```
# Adicionar suporte a arquitetura 32-bit no sistema 64-bit
sudo dpkg --add-architecture i386

# Baixar e adicionar a key
wget -O - https://dl.winehq.org/wine-builds/winehq.key | sudo apt-key add -

# Agora você precisa adicionar o repositório correspondente a sua distribuição, darei o exemplo de como fiz no meu
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/focal/winehq-focal.sources

# Atualizar a lista de pacotes com o novo repositório
sudo apt update

# Instalar a versão estável do Wine
sudo apt install --install-recommends winehq-stable -y
```

Pronto, se você quiser ser prevenido(a), pode instalar essas dependências também.

```
# Instalar versões 32-bit das bibliotecas essenciais
sudo apt install -y wine32          # Suporte a 32-bit no Wine
sudo apt install -y libgnutls30:i386 # Suporte a criptografia (ntdll)
sudo apt install -y libgcc-s1:i386  # Biblioteca GCC 32-bit

# Instalar winetricks para gerenciar DLLs e componentes
sudo apt install -y winetricks

# Instalar bibliotecas de desenvolvimento (úteis para compilação)
sudo apt install -y libwine-dev
``` 

Com o Wine instalado você consegue rodar o Ida Pro, x64dbg e ler arquivos executáveis windows (PE32).

### Editor Hex

Para essa etapa eu recomendo que você baixe 2 editores diferentes. Por que? Por que um é bonito e intuitivo e funciona para 99% dos casos, exceto um pequeno detalhe na última aula. O outro você só vai utilizar para colar uma parte do código hex e colar sem sobrescrever o que já existia antes.

O melhor editor hex que encontrei foi o GHex, ele é o que você vai usar na maior parte do tempo, para o truque explicado pelo professor na última aula você pode utilizar o bless hex editor.

Segue imagens dos dois editores, o bless e o ghex respectivamente
![bless](/imagens/imagem1.1.png "bless")
![ghex](/imagens/imagem1.2.png "ghex")

Você pode instalar ambos por flatpak ou:

```
sudo apt install ghex
sudo apt install bless
```

### Qemu

O Qemu (Quick Emulator) tem o nome auto explicativo. Ele é um emulador simples que iremos utilizar no lugar do VMWare, você pode tentar usar o VMWare, eu prefiro o Qemu por realmente achar ele mais simples e rápido, tanto na configuração quanto no uso.

```
# Instalar o QEMU para arquitetura i386/x86
sudo apt install qemu-system-x86 -y

# Utilitários adicionais do QEMU
sudo apt install qemu-utils -y

# Suporte a debugging, vai ser útil para usar com o Ida Pro
sudo apt install qemu-system -y  # Pacote meta com todos os sistemas

# Nas primeiras aulas, quando você só precisar testar o código Assembly para ver se está escrevendo corretamente na tela, use o comando
qemu-system-i386 >nomedoarquivo.bin<

# Comando para executar com o debugger, no Ida Pro você vai configurar igual o Rafa mostra na aula
qemu-system-i386 -s (ou gdp tcp:::port) -S -fda >nomedoarquivo.bin<
```

Abaixo uma explico para quê serve cada argumento:

* -S para debug, qemu vai iniciar mas não vai executar a primeira instrução
* -fda (aka First Disk Drive A), serve para carregar o arquivo passado no argumento como floppy disk
* -gdp tcp::port serve para especificar a porta que o debugger vai rodar, pode ser substituído por -s (que é um atalho para -gdb tcp::1234)

### Pinta

É, esse tem um nome diferente, eu sei. É simples: você vai usar no lugar do paint. Por que esse em específico? Porque nesse você consegue editar os pixeis da largura x altura facilmente e isso vai ser útil na hora de criar sua imagem BMP. Você pode instalar o [Pinta](https://www.pinta-project.com/howto/installing-pinta/) pelo Flatpak, Snap Store ou pela CLI.


```
# Adicionar o repositório
wget https://launchpad.net/~xtradeb/+archive/ubuntu/apps/+files/xtradeb-apt-source_0.4_all.deb
sudo apt install ./xtradeb-apt-source_0.4_all.deb
rm ./xtradeb-apt-source_0.4_all.deb

# Atualize a lista de pacotes
sudo apt-get update

# Instale o Pinta
sudo apt-get install pinta
```

### ImageMagick

Esse é bem simples, no Windows na hora de salvar como você pode escolher que tipo de BMP você vai salvar, no linux não. Então você vai usar o [ImageMagick](https://imagemagick.org/#gsc.tab=0) para converter o arquivo BMP para BMP monocromático. 


```
# Instalando
sudo apt install imagemagick

# Convertendo para BMP mono
convert imagem_original.png -monochrome imagem_monocromatica.bmp
```

### mingw-w64

Esse você vai utilizar basicamente para ler um arquivo executável no x64dbg (nesse caso x32dbg porque vai ser um arquivo 32bits). Você compilando com GCC vai gerar um executável Linux, e com esse programa você vai conseguir compilar um executável Windows (PE32) para poder debugar no x64dbg.

```
# Instalando
sudo apt install wingw-w64

# Compilar para Windows 32-bit (PE32)
i686-w64-mingw32-gcc bmp.c main.c -o leitorbmp.exe
```

## Segundo passo: fazendo algumas modificações.

Essa parte é a mais simples, precisaremos fazer algumas adaptações para que tudo ocorra bem.

### Pinta

No pinta, você vai apertar CTRL + R e selecionar a altura e largura da imagem conforme indicado no curso. Lembre de desmarcar a opção "Manter a taxa de proporção" e marcar a opção "Por tamanho absoluto:" para que possa escolher a quantidade de pixels.
![Como fazer o recorte no Pinta](/imagens/imagens2.png "Como fazer recorte no Pinta.")

### Ida Pro

No Ida, você precisa mover a janela General Register para uma janela flutuante e ir alterando suas dimensões para que ela fique legível, é chato, mas é o melhor que dá para fazer. Como você está rodando um programa para Windows no Linux através do Wine acontece esses bugs na interface.

Assim ficará o seu Ida Pro com essa modificação:
![idapro](/imagens/imagem3.png "ida pro")

### Encontrar a Main

Você vai precisar encontrar o endereço da função main no arquivo executável do seu leitor de BMP. Como você está usando o x32dbg no Wine, ele não vai carregar as funções normalmente e você vai precisar achar sua main manualmente para poder criar seu breakpoint, foi assim que consegui:

```
# Compilando e criando um arquivo Map
i686-w64-mingw32-gcc -g -m32 bmp.c main.c -o leitorbmp_debug.exe -Wl,-Map=output.map

# Encontrando a main com grep
grep "main" output.map

# O output é gigantesco, mas você deve encontrar algo como: 
0x0000000000401864                main
```

### Qemu

Nas primeiras aulas, ao iniciar o Qemu com o comando ```qemu-system-i386 >nomedoarquivo<.bin``` você vai se deparar com uma mensagem chata da SeaBIOS que pode acabar te atrapalhando.

![qemu1](/imagens/imagem4.1.png "qemu1")

Para isso você pode utilizar a interrupção 2 que nada vai alterar seus resultados e assim manter a tela do Qemu limpa. No começo do código assembly você vai escrever

```
mov ax, 0x02
int 0x10
```

E após inserir essa interrupção, ficará assim:

![qemu2](/imagens/imagem4.2.png "qemu2")

Quando o Rafa começar a usar qualquer outra interrupção essa mensagem da SeaBIOS já não vai aparecer.

## Observações

Ao decorrer do módulo você vai perceber alguns erros de compilação em C, isso acontece porque o Rafa está utilizando o Visual Studio que só existe para Windows e uma ou outra biblioteca C que também é exclusiva do Windows. Para contornar essa situação você vai precisar fazer algumas modificações no seu código C. Existem diversas maneiras de contornar essa situação e eu deixarei para que você, caro estudante, descubra-as sozinho(a). 

Como não se trata da impossibilidade de realizar algo porque lhe falta uma ferramenta no seu OS, aqui se encerra o propósito desse repositório. Não tema; não é nada absurdo e você conseguirá fazer essas modificações sem muita dificuldade. Inclusive, esse momento de ter que improvisar algo no código vai ser útil para você no futuro.

Futuramente, conforme eu for concluindo os módulos posso trazer outras ferramentas ou modificações para ajudar os alunos que utilizam GNU/Linux.

Espero que esse repo seja útil para vocês. Até mais! 😁
