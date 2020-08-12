# easy-but-not-infallible-way-to-protect-steam-games
A rapid / easy way to add some layer of protection to Godot+Steam games


## Introducción
El día 03/08/2020 y tras 2 años de trabajo, publiqué mi primer juego en Steam, [Cursed Gem](https://store.steampowered.com/app/1194480/Cursed_Gem/). Desgraciadamente el juego terminó publicado en páginas piratas de cracks/torrents a las 24h. Quiero recalcar que si mi juego ha sido pirateado, significa que es lo suficientemente atractivo como para que alguien se tome la molestia en piratearlo. Además hay varios artículos que hablan sobre como este fenómeno puede traducirse como una campaña de márketing y dar mayor visibilidad a pequeños desarrolladores indie como yo.

Este artículo no pretende debatir sobre si es bueno o no que pirateen tu software, o si es bueno o no el uso de DRM, o si es bueno o no permitir/denegar el acceso a tu juego pirata. Simplemente quiero compartir un método sencillo de implementar, que añade una pequeña capa de protección a tu juego para detectar si el jugador lo ha comprado o lo ha pirateado. En ese momento es decisión tuyo de implementar la acción que más desees, por ejemplo mostrar un amigable mensaje o directamente bloquear el juego. Empezamos?
