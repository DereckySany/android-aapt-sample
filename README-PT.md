# AAPT e aapt2 -- ID de recurso fixo e tag pública
***2021-07-20 13:39:11 POR STVEN_ KING***

## Prefácio
O artigo inteiro é sobre os pontos tinker de TinkerResourceIdTask conhecimento no .

- `aapt` e `aapt2` A diferença de Ambiente de execução e resultados de execução;
- `id` Fixação de recursos ;
- Conduzir PUBLIC A etiqueta:
    * `aapt` O ambiente operacional é gradle:2.2.0 e gradle-wrapper:3.4.1
    * `aapt2` O ambiente operacional é gradle:3.3.2 e gradle-wrapper:5.6.2

android-aapt-sample O projeto é um exemplo do meu próprio experimento. Sim `aapt` e `aapt2` Dois ramos, Correspondendo à sua implementação respectivamente.

## Resumo da AAPT
a partir do Android Studio 3.0 início, google o padrão é em `aapt2` Como compilador para compilação de recursos, `aapt2` Aparência, 
Fornece suporte para compilação incremental de recursos. Claro, haverá alguns problemas no processo de usar ,
Podemos passar em `gradle.properties` Configuração do meio android.enableAapt2=false Para fechar `aapt2`.

## Recursos
`Android` Nascido para ser compatível com uma ampla gama de dispositivos diferentes para fazer muito trabalho, como tamanho da tela,
internacionalização, teclado, densidade de pixels e assim por diante, podemos fazer compatibilidade para uma variedade de cenários específicos usando recursos específicos sem alterar uma linha de código , 
Suponha que adaptemos diferentes recursos para diferentes cenários , Como podemos aplicar esses recursos rapidamente `Android` Para nós `R` Esta classe , 
Especifica o índice de um recurso `id`, Então precisamos apenas informar ao sistema que em diferentes cenários de negócios , 
Basta usar o recursos correspondentes, Quanto ao arquivo específico no recurso especificado, é decidido pelo sistema de acordo com a configuração do desenvolvedor.


Neste caso , Suponha que demos valor `id` sim `x` , Agora quando o negócio precisa usar este recurso , O estado do telefone é `y` valor , Com ( `x`,`y`), 
Em uma tabela, você pode localizar rapidamente o caminho específico do arquivo de recurso . Este relógio é `resources.arsc`, é de `aapt` compilado .

Na verdade, recursos `binários` Como a `imagem` Não há necessidade de compilar, é apenas esse ato de “compilar”, 
é gerar `resources.arsc` e correto a `xml` operação binária do arquivo, `resources.arsc` é o relógio acima, 
`xml` a binarização de é para melhor desempenho na leitura do sistema . 
AssetManager Depois que chamamos `R` dependente `id` Quando, Você encontrará o arquivo correspondente nesta tabela, Leia.

`Gradle` No processo de compilação de recursos, é o que chamamos de comando `aapt2` , os parâmetros também são descritos neste documento, 
está apenas ocultando os detalhes da chamada dos desenvolvedores.

`aapt2` Existem principalmente dois passos, um passo é compile, um passo é `link`.

Crie um projeto vazio Apenas dois `xml`, Nomeadamente `AndroidManifest.xml` e `activity_main.xml`.

## Compilar
```
 mkdir compiled
 aapt2 compile src/main/res/layout/activity_main.xml -o compiled/
 Copy code 
```
fique compiled na pasta , Gerado `layout_activity_main.xml.flat` Este arquivo , É `aapt2` Peculiar , `aapt` Não, ( `aaptA` cópia é o arquivo fonte ), 
`aapt2` Pode ser usado para compilação incremental . Se tivermos muitos documentos, você precisa chamar... Por sua vez `compile` Talento, Na verdade, 
também pode ser usado aqui `–dir` Parâmetros, Só que este parâmetro não tem efeito de compilação incremental. 
em outras palavras, ao passar o diretório inteiro, mesmo que apenas um recurso tenha sido alterado, 
`AAPT2` ele também recompilará todos os arquivos do diretório.

## Link

link Taxa de carga de trabalho de compile Um pouco mais , A entrada aqui é múltipla flat O arquivo de e AndroidManifest.xml, Recursos externos , 
A saída é somente recurso apk e R.java. A ordem é a seguinte:
```
aapt2 link -o out.apk \
-I $ANDROID_HOME/platforms/android-28/android.jar \
compiled/layout_activity_main.xml.flat \
--java src/main/java \
--manifest src/main/AndroidManifest.xml
 Copy code 
```

- A segunda linha -I sim import Recursos externos , Aqui está principalmente android Algumas propriedades definidas sob o namespace , Geralmente usamos @android:xxxÉ tudo neste jar Inside , Na verdade, também podemos fornecer nossos próprios recursos para outros vincularem ;
- A terceira linha é o flat arquivo de entrada , se houver mais de um , fica logo atrás ;
- A quarta linha é R.java Diretório gerado;
- A quinta linha especifica AndroidManifest.xml;

## Link
Quando terminar, irá gerar out.apke `R.java`, `out.apk` Contém um `resources.arsc` arquivo . 
Somente com arquivo de recurso pode-se usar sufixo `.ap_`.

## Veja os recursos compilados
Além de usar Android Studio Para visualizar o resources.arsc, também pode ser usado diretamente aapt2 dump apk Informações para visualizar o status e o recurso relacionado ID ：
```
aapt2 dump out.apk
 Copy code 
```
A saída é a seguinte:
```
Binary APK
Package name=com.geminiwen.hello id=7f
  type layout id=01 entryCount=1
    resource 0x7f010000 layout/activity_main
      () (file) res/layout/activity_main.xml type=XML
 Copy code 
 ```
Você pode ver `layout/activity_main` Correspondente `ID` em`0x7f010000`.

## Compartilhamento de recursos
`android.jar` É apenas uma pilha para compilar , Quando é realmente implementado , 
`Android OS` Fornece uma biblioteca de tempo de execução ( `framework.jar` ). `android.jar` É muito parecido com um `apk`, 
é só que existe `class` file , então existe um `AndroidManifest.xml` e `resources.arsc`. Isso significa que também podemos usá-lo `aapt2 dump`, 
execute o seguinte comando:
```
aapt2 dump $ANDROID_HOME/platforms/android-28/android.jar > test.out
 Copy code 
 ```
 Você obtém um monte de saída como esta:
```
resource 0x010a0000 anim/fade_in PUBLIC
      () (file) res/anim/fade_in.xml type=XML
    resource 0x010a0001 anim/fade_out PUBLIC
      () (file) res/anim/fade_out.xml type=XML
    resource 0x010a0002 anim/slide_in_left PUBLIC
      () (file) res/anim/slide_in_left.xml type=XML
    resource 0x010a0003 anim/slide_out_right PUBLIC
      () (file) res/anim/slide_out_right.xml type=XML
 Copy code 
 ```
 É um pouco mais `PUBLIC` Campo de , Um `apk` Os recursos no arquivo , Se estiver marcado com isso , Pode ser usado por outros `apk` Os referenciados , 
 A forma de citar é `@Packagename:type/name` , por exemplo `@android:color/red`.

Se quisermos fornecer nossos recursos , Bem, antes de tudo, estabeleça uma base para nossos recursos `PUBLIC` A tag , 
E então em `xml` Referencie o nome do seu pacote em , como  `@com.gemini.app:color/red` Você pode se referir ao que você define `color/red` , 
Se você não especificar o nome do pacote, o padrão é você mesmo.

### quanto a `AAPT2` Como gerar `PUBLIC`, Se você estiver interessado, leia este abaixo .

## resumo de ids.xml
`ids.xml` Forneça recursos exclusivos para recursos relacionados ao aplicativo `id`. `id` Trata-se de obter `xml` os parâmetros necessários para objetos em , 
ou seja, `Object = findViewById(R.id.id_name)`; Medium `id_name`.

Esses valores podem ser usados no código `android.R.id` Quote to . Se em `ids.xml` Isso define `ID` , está em `layout` 
Pode ser definido da seguinte `@id/price_edit` forma , caso contrário `@+id/price_edit`.

## | Avançado

- É fácil nomear, podemos nomear alguns controles específicos primeiro, referência direta ao usar `id` isso fará, uma etapa de nomenclatura é omitida.
- Otimize a eficiência da compilação:
    - add to `id` irá em in `R.java` ;
    - Use `ids.xml` o gerenciamento unificado, compile uma vez e use muitas vezes. Mas use `"@+id/btn_next"` Na forma de , 
      Toda vez que um arquivo for salvo (Ctrl+s) after `R.java` Será testado novamente , 
      Se o `id` Então não gerar , Se não existir, você precisa adicionar o `id`. Portanto, a eficiência de compilação é reduzida.
      
ids.xmlO conteúdo do documento:
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="forecast_list" type="id"/>
<!--    <item name="app_name" type="string" />-->
</resources>
 Copy code 
 ```
 Algumas pessoas podem estar curiosas que há uma linha de código anotado nele , Abra o comentário e você verá que o compilador reportará um erro.
 ```
> Execution failed for task ':app:mergeDebugResources'.
> [string/app_name] /Users/tanzx/AndroidStudioProjects/AaptDemo/app/src/main/res/values/strings.xml	
> [string/app_name] /Users/tanzx/AndroidStudioProjects/AaptDemo/app/src/main/res/values/ids.xml: Error: Duplicate resources
 Copy code 
 ```
 porque `app_name` os recursos para já estão `value` declarados em .
 
 ## resumo public.xml
 Instruções oficiais Site oficial Selecione os recursos que deseja tornar públicos .
 ```
 
Tradução original Todos os recursos da biblioteca são públicos por padrão. Para tornar todos os recursos implicitamente privados, você deve definir 
pelo menos um atributo específico como público. Os recursos incluem... Do seu projeto `res/` Todos os arquivos no diretório, como imagens. Para evitar que os usuários da biblioteca acessem recursos apenas para uso interno, você deve usar esse mecanismo automático de identificação privada declarando um ou 
mais recursos públicos. talvez, Você também pode adicionar uma `<public/>` Tag vazia torna todos os recursos privados, Esta tag não torna nenhum recurso  público, Vai levar tudo Todos os recursos Todos privados .

Tornando as propriedades implicitamente privadas, você pode não apenas impedir que os usuários da biblioteca obtenham conselhos de conclusão de código dos recursos internos da biblioteca, mas também renomear ou remover recursos privados, sem destruir o lado cliente da biblioteca. O sistema filtra recursos privados do preenchimento de código, além disso, um aviso Lint Um aviso é emitido quando você tenta fazer referência a recursos privados.

Ao construir a biblioteca, Android Gradle O plug-in obtém a definição de recurso público e extrai-o para `public.txt` In file, O sistema então empacota o arquivo em AAR In file .

```
Os resultados medidos estão apenas incompletos, está vermelho. Se realizado `lint Check` , Não há avisos para compilar

Agora a maior parte da explicação é arquivo `res/value/public.xml` Usado para colocar recursos fixos `ID` Atribuído a `Android` recursos.

`public.xml` O conteúdo do documento:
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <public name="forecast_list" id="0x7f040001" type="id" />
    <public name="app_name" id="0x7f070002" type="string" />
    <public name="string3" id="0x7f070003" type="string" />
</resources>
 Copy code 
 ```
 ## Correção de ID de recursos
 
recursos `id` A fixação de é extremamente importante em reparo a quente e plug-in. Em reparo a quente , estrutura `patch` quando , Precisa manter os recursos do `patch` pacote `id` E recursos do pacote de referência `id` Acordo ; No plug-in , se o plug-in precisar fazer referência aos recursos do host , você precisará transferir os recursos do host `id` Fix , portanto , recursos `id` É especialmente importante ser corrigido nesses dois cenários .

ficar Android Gradle Plugin 3.0.0em , O padrão é `aapt2` ativado , O `aapt` original O modo fixo de recursos `public.xml` Ele também falhará , Temos que encontrar uma nova maneira de corrigir recursos , Em vez de simplesmente desativá-lo `aapt2`, Então este artigo discute `aapt` and `aapt2` Como realizar o gerenciamento de recursos, respectivamente `id` Fixação.

## aapt id Fixação de Conduta:

Configuração do ambiente do projeto `PS` Faça reclamações sobre isso `aapt` Foi `aapt2` Em vez de ,`aapt` Há pouca informação sobre isso, É muito difícil construir o ambiente
```
com.android.tools.build:gradle:2.2.0

distributionUrl=https\://services.gradle.org/distributions/gradle-3.4.1-all.zip

compileSdkVersion 24

buildToolsVersion '24.0.0'
```
Primeiro em `value` Sob o arquivo, siga o acima `ids.xml` e `public.xml` E o nome do arquivo, Gere o arquivo correspondente.

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1de729a9c4b4b9094e3c350f580f3aa~tplv-k3u1fbpfcp-zoom-1.image)

Através da compilação direta do `R file` conteúdo de , você pode ver os recursos que queremos configurar. `id` Não gerou como esperávamos.

||levar `public.xml` a cópia do arquivo para o `build/intermediates/res/merged` diretório correspondente

```
afterEvaluate {
    for (variant in android.applicationVariants) {
        def scope = variant.getVariantData().getScope()
        String mergeTaskName = scope.getMergeResourcesTask().name
        def mergeTask = tasks.getByName(mergeTaskName)
        mergeTask.doLast {
            copy {
                int i=0
                from(android.sourceSets.main.res.srcDirs) {
                    include 'values/public.xml'
                    rename 'public.xml', (i++ == 0? "public.xml": "public_${i}.xml")
                }
                into(mergeTask.outputDir)
            }
        }
    }
}
 Copy code 
```
