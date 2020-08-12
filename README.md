# Introducción
El día 03/08/2020 y tras 2 años de trabajo, publiqué mi primer juego en Steam, [Cursed Gem](https://store.steampowered.com/app/1194480/Cursed_Gem/). Desgraciadamente el juego terminó publicado en páginas piratas de cracks/torrents a las 24h. Quiero recalcar que si mi juego ha sido pirateado, significa que es lo suficientemente atractivo como para que alguien se tome la molestia en piratearlo. Además hay varios artículos que hablan sobre como este fenómeno puede traducirse como una campaña de márketing y dar mayor visibilidad a pequeños desarrolladores indie como yo.

Este artículo no pretende debatir sobre si es bueno o no que pirateen tu software, o si es bueno o no el uso de DRM, o si es bueno o no permitir/denegar el acceso a tu juego pirata. Simplemente quiero compartir un método sencillo de implementar, que añade una pequeña capa de protección a tu juego para detectar si el jugador lo ha comprado o lo ha pirateado. En ese momento es decisión tuyo de implementar la acción que más desees, por ejemplo mostrar un amigable mensaje o directamente bloquear el juego. Empezamos?

# Algo que debes saber sobre los sistemas de protección
No existe ningún método 100% infalible que permita protejer tu juego/aplicación. Da igual lo complejo que sea el sistema, da igual la contidad de encriptación y ofuscación que utilices, da absolutamente igual. Cualquier sistema que implementes podrá ser crackeado/pirateado si caen en manos experimentadas y se le dedica el tiempo suficiente.

Los sistemas de protección de hoy en día se basan en complicar mucho la tarea de cracking, de tal forma que el crack aparezca varios días (o semanas!) después de la publicación del juego, consiguiendo que jugadores impacientes terminen comprando el juego en lugar de esperar el crack. De hecho es muy común que empresas grandes decidan actualizar y eliminar el DRM de sus juegos el día siguiente que haya sido crackeado. Estos sistemas de protección simplemente buscan ganar tiempo.

Dicho esto, no creo que sea necesario recordarlo, pero el sistema que he implementado en mi juego tampoco es infalible. Si un cracker le dedica tiempo, terminará pirateando el juego. La gracia de este sistema que voy a explicar es que el cracker no sabe que su "crack" no funciona, convirtiendo el juego pirateado en una versión "demo" del juego para que, con un poco de suerte, el jugador pirata decida comprarlo.

# Escenario, herramientas, víctima, protector y verdugo.
### El Escenario
Cursed Gem es un juego programado en [Godot Engine](https://godotengine.org/). No quiero entrar en detalles ya que el artículo se alargaría mucho, pero básicamente tenemos que saber que cuando exportamos un juego, lo que realmente estamos obteniendo es una copia del motor del juego (el engine, sin las herramientas de edición) y todo nuestro código fuente empaquetado en un fichero PCK. En otros motores de juego lo que normalmente obtenemos es una compilación, es decir, nuestro código fuente es linkado y compilado, generando un binario único que se puede ejecutar/jugar. Con Godot eso no ocurre ya que no existe compilación. GDScript es un lenguaje interpretado (como Python) y eso nos otorga muchas ventajas y debilidades.

Qué significa? Pues que es muy fácil revertir el proceso de empaquetado y obtener el código fuente a partir del fichero PCK. Si el cracker decide "atacar" nuestro juego desempaquetando el PCK, le será muy fácil descubrir nuestro sistema y podrá piratear de nuevo el juego. Por suerte, los crackers no suelen hacer este proceso ya que es laborioso. Solo recurren a él cuando sus herramientas básicas fallan por algún motivo.

Cuál será nuestro objetivo? Hacer creer que el cracker ha podido piratear nuestro juego con sus herramientas básicas, pero hacer saltar la protección/bloqueo tras varios minutos/horas de juego. Así el cracker no sabrá nunca que su crack no funciona y tampoco decidirá utilizar técnicas más avanzadas (como desempaquetar el PCK que he comentado antes.)

### Las herramientas
- Godot Engine: El sistema de protección se implementa directamente en el código de nuestro juego, así que nuestra herramienta principal será Godot Engine.
- Generador de checksums: Cualquier herramienta que nos permita calcular el checksum de un fichero, mi favorita es [HastTab](http://implbits.com/products/hashtab/) que se integra perfectamente en el explorador de ficheros de Windows.
- Steam SDK: El juego está publicado en Steam, y por tanto usamos su SDK/DLL

### La Víctima
La victima, por suerte o por desgracia, será nuestro juego. Al tratarse de un proyecto hecho en Godot, tenemos 2 ficheros que proteger, el EXE y el PCK. Nuestra protección se encargará de comprobar la integridad del engine (el fichero EXE) así como la integridad del SDK/DLL de Steam. Luego añadiremos "checks" adicionales que nos permitirán saber si el juego se ejecuta de forma legal o pirata. Todas estas comprobaciones harán que el cracker no pueda utilizar sus herramientas básicas y tenga que dedicar parte de su tiempo y esfuerzo en eliminar todas las protecciones. El éxito de este sistema reside en hacer creer que el cracker ha consegido piratear el sotware, para evitar que decida utilizar otras técnicas avanzadas.

### El protector
Queremos publicar nuestro juego en Steam, y por tanto hemos de utilizar su SDK oficial. Para que nuestro juego pueda "hablar" con Steam, haremos llamadas utilizando su DLL como pasarela. Por ejemplo si queremos desbloquear un logro, el juego simplemente cargará la librería de Steam y llamará a la función `Steam.setAchievement("achievement_example")`, si queremos comprobar si Steam está ejecutándose en el PC podemos llamar a la función `Steam.loggedOn()`, o si queremos comprobar el ID del jugador de Steam, podemos llamar la función `Steam.getSteamID()`.

Todas estas funciones están ya programadas e integradas en una DLL que Steam nos ofrece llamada `steam_api.dll` o `steam_api64.dll`, el listado de funciones disponibles lo tenemos [aquí](https://partner.steamgames.com/doc/api). Este es nuestro protector, ya que podemos de una forma fácil y rápida comprobar si el usuario actual tiene Steam abierto, y si lo tiene, comprobar si posee (ha comprado) nuestro juego. Dicha comprobación se obtiene a través de la función [BIsSubscribed](https://partner.steamgames.com/doc/api/ISteamApps). Os dejo un ejemplo:

![](https://i.imgur.com/oWSeqzQ.png)  
*source: https://gramps.github.io/GodotSteam/tutorials-initializing.html*

### El verdugo
Para un cracker, lo más sencillo es interceptar las llamadas que hace tu juego sobre la API de Steam, modificando la respuesta.
¿Qué ocurriría si un cracker consigue que la DLL de Steam devuelva `true` siempre a la pregunta de `Steam.IsSubscribed()`? Pues que tu juego pensaría que el usuario/jugador posee una copia legítima en Steam, ya que la DLL siempre devolvería `true`. Bien pues esta técnica es la herramienta principal de los crackers hoy en día. Pero... cómo lo consiguen? Muy fácil, intercambiando la DLL oficial de Steam por una DLL modificada. Veamos un ejemplo real...

# Emuladores de Steam, la navaja suiza de los crackers
Este es el método más usado hoy en día para piratear juegos de Steam. En internet existen implementaciones/copias de la DLL oficial de Steam `steam_api64.dll` ligeramente modificadas para que ciertas funciones devuelvan siempre el mismo resultado sin preguntar a los servidores oficiales de Steam. Estas implementaciones de la DLL se conocen como "emuladores". Haciendo que, por ejemplo, siempre devuelvan `true` cuando el juego llama a la función `Steam.IsSubscribed()`. El cracker simplemente tiene que hacer lo siguiente:

- Comprar el juego oficial en Steam.
- Descargar el juego e instalarlo.
- Crear una copia de los ficheros/juego en otra carpeta.
- Cambiar el fichero `steam_api64.dll` oficial por el `steam_api64.dll` crackeado del emulador.
- Comprobar que el juego se ejecuta con la DLL pirata.
- Desinstalar el juego.
- Devolver el juego (refund) para recuperar el dinero.
- Subir un ZIP con la copia del juego + DLL pirata a una web de torrents.

Uno de los emuladores más utilizados actualmente es el [SteamEmu de Goldberg](https://gitlab.com/Mr_Goldberg/goldberg_emulator), no estoy dando información secreta ni mucho menos. Cualquiera que descargue un juego pirata y examine la DLL verá que pone Goldberg. Es información ampliamente conocida en los foros de internet.

Para este ejemplo tenemos el código fuente oficial de la DLL pirata, y podemos comprobar como han re-implementado la función `Steam.IsSubscribed()`:

    bool Steam_Apps::BIsSubscribed()
    {
        PRINT_DEBUG("BIsSubscribed\n");
        return true;
    }

Imprime una línea de debug y devuelve true!!! independientemente de que el usuario haya comprado o no el juego. Otras muchas funciones de la DLL oficial han sido modificadas, como por ejemplo la compra de DLCs. Esto es un verdadero horror, ya que cambiando la DLL oficial por la DLL del emulador consiguen que el propio juego no sepa que está siendo pirateado.

# Identificando una copia pirata
Tenemos diferentes formas para ello. Podemos comprobar la ruta/path del juego, comprobar los argumentos adicionales de ejecución, comprobar la existencia de ciertos ficheros o carpetas, etc... pero empezaremos por lo más obvio: Que el juego detecte si el fichero `steam_api64.dll` es el oficial o una modificado.

### Comprobando la autenticidad de steam_api64.dll
En informatica y en el mundo de las telecomunicaciones existe algo llamado [checksum](https://en.wikipedia.org/wiki/Checksum). Básicamente es una función que realiza cálculos matemáticos sobre una serie de datos y devuelve un resultado. Un mismo fichero siempre tiene el mismo resultado, si cambiamos una sola letra de ese fichero, el resultado de su checksum será diferente. De esta forma es muy fácil garantizar la integridad de una información o fichero.

Existen varias funciones checksum, no voy a entrar en detalles, pero las más utilizadas con CRC32, MD5 y SHA256. Lo primero que haremos será calcular el checksum de la DLL oficial:

![](https://i.imgur.com/oMTuXmM.png)

El checksum SHA256 de `steam_api64.dll` oficial es: `A178F19A516023EFC3CC30B1E90FBEDA838D08D4F1FA006B895608D67FA60EAC`
Si calculamos ahora el SHA256 de la DLL pirata de Goldberg, obtendremos un checksum muy diferente: `43C19ECECD799332CC09A56A11A99E90E1FC884061B046F9D1E7203330A3B721`

Con una simple función podemos calcular el SHA256 de la DLL del juego y si no coincide con `A178F19A516023EFC3CC30B1E90FBEDA838D08D4F1FA006B895608D67FA60EAC` sabremos indudablemente que el juego es pirata. En ese momento levantaremos un flag para, horas más tarde, mostrar una advertencia o bloquear el juego. Os dejo un ejemplo en Godot:

    func check1() -> bool:
	    # piracy flag
        var yar = false
	    var file = File.new()
	
	    if file.file_exists("./steam_api64.dll"):
		    if file.get_sha256("./steam_api64.dll") != "a178f19a516023efc3cc30b1e90fbeda838d08d4f1fa006b895608d67fa60eac":
			    yar = true
	    else:
		    yar = true
	    return yar

### Comprobando la existencia de ficheros/carpetas no oficiales
Después de investigar sobre el funcionamiento del emulador de Goldberg, descubrí que suele tener un fichero llamado `local_save.txt` y una carpeta llamada `steam_settings`. No es obligatorio que el juego pirata tenga esos ficheros pero el 100% de juegos piratas que he visto lo tienen, así que siempre es bueno añadir esta comprobación. Os dejo un ejemplo de mi juego "real/oficial" y otra foto del juego pirateado para que veais las diferencias y los ficheros:

![](https://i.imgur.com/famnOSU.png)  
Carpeta real. No exite `local_save.txt` ni `steam_settings`. El tamaño de la DLL es pequeño, 257Kb

![](https://i.imgur.com/CCVsIUq.png)  
Juego pirata! Existe el fichero `local_save.txt` y la carpeta `steam_settings`. La DLL está modificada, su tamaño es enorme!! Aunque eso lo estamos verificando con el checksum, en el punto anterior.

Sabiendo esto, podemos añadir nuevas comprobaciones dentro del código del juego:

    func check2() -> bool:
        # piracy flag
	    var yar = false
	    var file = File.new()
	    var dir = Directory.new()

	    if dir.open("./steam_settings") == OK:
		    yar = true
	
	    if file.file_exists("./steam_api64.dll"):
		    yar = true
         return yar



### Comprobando los argumentos adicionales de ejecución

### Proteger la integridad del motor
