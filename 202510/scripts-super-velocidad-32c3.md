---
Title: Scripts: Super speed [ES]
Description: 
Author: Matthew Ramirez
Date: 2025-10-26T21:47:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>¡Hola! Esta es mi primera publicación y hoy aprendí a crear un script de Super velocidad. El personaje podrá aumentar su velocidad al mantener presionada la tecla Shift.</p>

<h2>
  
  
  ¿Dónde debemos colocar nuestro Script?
</h2>

<p>Debe ubicarse cerca del personaje, dentro de StarterPlayer -&gt; StarterCharacterScripts. De esta forma, nuestro script tendrá contexto sobre los hijos de Character, como el Humanoid.</p>

<p>Usaremos UserInputService para manejar las entradas del usuario y actualizar la WalkSpeed ​​del Humanoid.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>local humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")
local defaultWaklSpeed = humanoid.WalkSpeed

local UserInputService = game:GetService("UserInputService")

UserInputService.InputBegan:Connect(
    function(input, processed)
        if not processed and input.KeyCode == Enum.KeyCode.LeftShift then
            print("Se esta presionando shift")

            humanoid.WalkSpeed = defaultWaklSpeed * 2
        end
    end
)

UserInputService.InputEnded:Connect(
    function(input)
        if input.KeyCode == Enum.KeyCode.LeftShift then
            print("Se dejo de presionar shift")

            humanoid.WalkSpeed = defaultWaklSpeed
        end
    end
)
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzuvkhsphmbrepw353jli.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzuvkhsphmbrepw353jli.gif" alt=" " width="426" height="240"></a></p>

<p>Gracias por leer, diviértanse!</p>

