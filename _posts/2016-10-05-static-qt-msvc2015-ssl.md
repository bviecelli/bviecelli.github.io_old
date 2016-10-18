---
title: Qt 5.6 estático + VS + SSL
featured: images/guide-tag.png
layout: post
---

# Compilando o Qt 5.6 estático, com o Visual Studio e com suporte à SSL

Se você precisa compilar suas aplicações feitas em Qt em um único executável sem depender um monte de DLL para que funcione em outros computadores você já deve ter pesquisado e descoberto que precisa compilar o código fonte do Qt com a opção *-static*.
Segui [este tutorial][tutorial] do blog do [Amin], compilei com sucesso as bibliotecas e quando fui compilar meu software que utiliza a classe QNetworkAccessManager conectando à um enredeço via https, descobri que não havia suporte à SSL pois na configuração da compilação do Qt estava a opção *-no-openssl*. Recomecei o processo com a opção *-openssl* e aí que os problemas começaram a aparecer.

## Solução

Depois de várias compilações e muitas pesquisas, resolvi o problema seguindo os seguintes passos:

### Dependências

Obviamente você já deve ter instalado alguma versão do Qt em seu computador, se não tem, instale através do [instalador online][qt-online] antes de prosseguir. Verifique também as recomendações do próprio Qt [aqui][qt-msvc-build].

- Visual Studio - Eu utilizei o Visual Studio Community que hoje é o MSVC2015 e pode ser baixado gratuitamente através do [site][vs-commu].
- [Source Package do Qt] - A versão em questão é a 5.6 mas você pode compilar outras versões também desde que você verifique as diferenças entre a compilação das mesmas.
- [Python] - Necessário para os scripts da compilação do Qt
- [Pearl] - Necessário para configurar o Makefile do Qt. Eu utilizei o Strawberry Pearl, mas creio que funcione com o Active Pearl também.
- OpenSSL - Se você tiver empenho e tempo para compilar o OpenSSL você vai sofrer um pouco(ou não) e provavelmente vai obter o mesmo resultado utilizando os pacotes prontos que encontrei [aqui][openssl-slproweb].

Extraia o arquivo *qt-everywhere-opensource-src-5.6.0.zip* para a pasta onde o Qt foi instalado, por exemplo na pasta:

> C:\Qt\5.6

### Configurando o ambiente e compilando

Para que não ocorram falhas nas etapas, após a instalação das dependencias citadas, você deve configurar os caminhos das mesmas na variável *PATH*, por exemplo:

> PATH=C:\ProgramData\Oracle\Java\javapath;C:\Program Files (x86)\Common Files\Intel\Shared Files\cpp\bin\Intel64;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files (x86)\ATI Technologies\ATI.ACE\Core-Static;C:\Program Files\Microsoft SQL Server\110\Tools\Binn\;C:\MinGW\bin;C:\MinGW\MSYS\1.0\bin;C:\Program Files\TortoiseSVN\bin;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem; **D:\Qt\Tools\QtCreator\bin**;**C:\Strawberry\c\bin**;**C:\Strawberry\perl\site\bin**;**C:\Strawberry\perl\bin**;**C:\OpenSSL-Win32\bin**;**C:\OpenSSL-Win32\include**; **C:\OpenSSL-Win32\lib\VC**;**C:\Python27**;C:\Program Files\MySQL\MySQL Server 5.6\bin;C:\Users\Bruno\.dnx\bin;C:\Program Files (x86)\Windows Kits\8.1\Windows Performance Toolkit\;C:\Program Files\Microsoft SQL Server\130\Tools\Binn\;C:\Program Files\Git\cmd;C:\Users\Bruno\.dnx\bin

Abra um prompt de comando e configure o ambiente de compilação do Visual Studio executando o comando abaixo, assumindo que o Visual Studio seja o 2015 Community e esteja instalado no diretório padrão.

> C:\WINDOWS\system32>"C:\Arquivos de Programas (x86)\Microsoft Visual Studio 14.0\VC\bin\vcvars32.bat"

Após isso verifique a variável *PATH* novamente digitando, no prompt, o comando

> path

Confira se os endereços que foram marcados em negrito estão na *PATH* antes de continuar e se não estiverem você pode digitar o seguinte comando para reconfigurar:

> set PATH=%PATH%;D:\Qt\Tools\QtCreator\bin;C:\Python27;C:\Strawberry\perl\bin

Então, você está, teoricamente, apto à compilar o Qt.

Acesse a pasta do código extraído do Qt

> cd C:\Qt\5.6\qt-everywhere-opensource-src-5.6.0

Configure esta variável de ambiente

> set QMAKESPEC=win32-msvc2015

E rode o seguinte comando para configurar a compilação

> configure.bat -static -debug-and-release -prefix "C:\Qt\5.6\msvc2015_static" -platform win32-msvc2015 -qt-zlib -qt-pcre -qt-libpng -qt-libjpeg -qt-harfbuzz -qt-freetype -opengl desktop -qt-sql-sqlite -qt-sql-odbc -openssl-linked -I C:\OpenSSL-Win32\include -L C:\OpenSSL-Win32\lib\VC OPENSSL_LIBS="-lUser32 -lAdvapi32 -lGdi32" OPENSSL_LIBS_DEBUG="-lssleay32MDd -llibeay32MDd" OPENSSL_LIBS_RELEASE="-lssleay32MD -llibeay32MD" -opensource -confirm-license -make libs -nomake tools -nomake examples -nomake tests

Indiquei que a pasta de instalação será *"C:\Qt\5.6\msvc2015_static"*. Observe que também indiquei a opção *-openssl-linked*, os *includes*, os caminhos onde estão e quais são as bibliotecas que instalei do pacote OpenSSL. Caso você opte por compilar os pacotes OpenSSL você deverá trocar os nomes e caminhos conforme sua saída.

Agora voce roda os seguintes comandos para compilar e instalar.

> jom
> jom install

Caso observem algum erro e percebeu como resolver o ideal é que você limpe a compilação anterior e comece de novo

> jom clean

É isso aí, se compilou e instalou sem erros você deve configurar o seu novo compilador no QtCreator e adicionar no seu arquivo .pro o seguinte parâmetro

> CONFIG += static

Compile e rode sua aplicação sem depender das DLLs.

   [tutorial]: <https://amin-ahmadi.com/2015/07/03/how-to-build-qt-5-5-static-libraries-using-any-microsoft-visual-c-compiler/>
   [amin]: <https://amin-ahmadi.com/>
   [python]: <https://www.python.org/downloads/release/python-2710/>
   [pearl]: <http://strawberryperl.com/>
   [qt-msvc-build]: <https://wiki.qt.io/Building_Qt_Desktop_for_Windows_with_MSVC>
   [openssl-slproweb]: <http://slproweb.com/download/Win32OpenSSL-1_0_2g.exe>
   [vs-commu]: <https://go.microsoft.com/fwlink/?LinkId=691978&clcid=0x416>
   [qt-online]: <http://download.qt.io/official_releases/online_installers/qt-unified-windows-x86-online.exe>
   [Source Package do Qt]: <http://download.qt.io/official_releases/qt/5.6/5.6.0/single/qt-everywhere-opensource-src-5.6.0.zip>

