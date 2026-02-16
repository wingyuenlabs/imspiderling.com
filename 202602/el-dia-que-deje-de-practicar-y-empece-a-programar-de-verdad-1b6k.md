---
Title: 🚀 El día que dejé de 'practicar' y empecé a programar de verdad
Description: 
Author: Laura Montironi
Date: 2026-02-16T21:20:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hay un momento en todo bootcamp en el que las excusas se terminan. Ya no vale el "estoy haciendo un ejercicio" o "es solo para clase". Llega el <strong>Proyecto Final</strong> y, de repente, la cosa se pone seria.</p>

<p>Después de meses peleándome con el DOM, entendiendo Angular, construyendo APIs con FastAPI y rompiendo cosas (sí, incluso en producción 😅), tocó levantar <strong>UpgradeFood</strong>. Y no ha sido precisamente un paseo por el campo.</p>

<p><strong>🧠 El ecosistema: Lo que hay bajo el capó</strong></p>

<p>Este no es el típico "Hola Mundo". He montado una arquitectura completa donde cada pieza tiene que hablarse con la otra sin explotar en el intento:</p>

<p><strong>Frontend:</strong> Angular (con sus Signals y estados que me han dado la vida).</p>

<p><strong>Backend:</strong> FastAPI (Python puro al servicio del caos).</p>

<p><strong>Base de Datos:</strong> MySQL viviendo en <strong>Railway</strong>.</p>

<p><strong>Imágenes:</strong> Todo gestionado en la nube con <strong>Cloudinary</strong>.</p>

<p><strong>Emails:</strong> Confirmaciones reales con <strong>Resend</strong>.</p>

<p><strong>Deploy:</strong> Backend en Railway y Frontend en <strong>Firebase</strong>.</p>

<p><strong>💥 Storytime: El día que borré el repositorio (y casi mi salud mental)</strong></p>

<p>Sí, pasó. <strong>Borré el repo del backend en GitHub.</strong> Con el backend ya desplegado, la base de datos conectada y todo funcionando... de repente, puf, desaparece. Ese silencio que se queda en la habitación es indescriptible. Pero ahí aprendí la lección de oro: <strong>Git no es opcional, es supervivencia.</strong></p>

<p>Gracias a que lo tenía en local y a que el deploy en Railway no se inmuta si borras el origen, pude reconstruirlo todo: reconfigurar variables de entorno, conectar remotos y volver a la vida. No es el error lo que te define, es cómo lo levantas cuando todo arde.</p>

<p><strong>🛠️ Romper para aprender (Mi especialidad)</strong></p>

<p>He crasheado el backend más veces de las que quiero admitir. ¿Mis "grandes éxitos"?</p>

<p>Cambiar un nombre de archivo y cargarme todos los imports.</p>

<p>Tener el <strong>CORS</strong> mal configurado y que Angular me ignorara sistemáticamente.</p>

<p>Variables de entorno que "juraría" que estaban bien... pero no.</p>

<p>El drama de los <strong>emails</strong>: Descubrir que en modo testing solo me llegaban a mí porque no había "verificado el dominio". Spoiler: No siempre es culpa de tu código, a veces es la infraestructura.</p>

<p><strong>🧩 Lo que realmente he aplicado (y funciona)</strong></p>

<p>Más allá de los sustos, <strong>UpgradeFood</strong> hace cosas muy potentes:</p>

<ol>
<li><p><strong>Auth Real:</strong> Registro, Login y protección de rutas con <strong>JWT</strong>. Si no tienes el token o no perteneces a la "familia", no pasas.</p></li>
<li><p><strong>Gestión de Reservas:</strong> Validar disponibilidad, roles de admin vs cliente y control total de la base de datos.</p></li>
<li><p><strong>Feedback Visual:</strong> Nada de alertas feas; he usado SweetAlert para que el usuario sepa qué está pasando.</p></li>
<li><p><strong>Tipado Real:</strong> He dicho adiós al any. Si algo se rompe, prefiero que me lo diga TypeScript antes que el usuario.</p></li>
</ol>

<p><strong>🎓 Lo que nadie te dice del proyecto final</strong></p>

<p>El código es solo el 50%. El resto es:</p>

<p>Gestionar la frustración cuando algo que funcionaba ayer, hoy no.</p>

<p>Aprender a debuggear sin entrar en pánico (bueno, con un poco de pánico).</p>

<p>Entender que <strong>"funciona en mi máquina"</strong> no sirve de nada si no funciona en el deploy.</p>

<p>He pasado de hacer ejercicios sueltos a entender cómo se habla un frontend en Firebase con un backend en Railway. He pasado de tener miedo a tocar la API, a romperla a propósito para entender por qué fallaba.</p>

<p><strong>💬 Conclusión</strong></p>

<p>Si algo me llevo de estas semanas es que no necesitas saberlo todo para empezar algo grande. Necesitas <strong>bases sólidas, mucha paciencia y la capacidad de volver a empezar.</strong></p>

<p>Hoy ya no siento que estoy "aprendiendo a programar". <strong>Estoy programando.</strong> Y aunque me ha pasado de todo, aquí sigo. Construyendo.</p>

