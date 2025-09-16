---
Title: Como uma Ligação Falsa do Suporte Google e 2FA Sincronizado na Nuvem Custou US$130.000 a um Desenvolvedor
Description: 
Author: Mr Punk da Silva
Date: 2025-09-16T21:15:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Em junho de 2025, o engenheiro de design David Scoville atendeu a uma ligação que parecia ser do suporte oficial do Google. Em poucos minutos, ao fornecer um “código de verificação” por telefone, ele perdeu o acesso à conta Google, às chaves do Authenticator em nuvem e, por fim, US$80 000 em ETH stakeados, equivalentes a aproximadamente US$130 000 hoje, após ter sua conta Coinbase esvaziada.</p>

<p>📰 Cobertura no CyNews: <a href="https://cynews.vercel.app/show/45264726" rel="noopener noreferrer">https://cynews.vercel.app/show/45264726</a><br><br>
📎 Texto original: “I Was Scammed Out of \$130,000—And Google Helped It Happen” by David Scoville (Substack)</p>




<h2>
  
  
  Como o golpe aconteceu
</h2>

<ul>
<li>
<p>A ligação  </p>

<ul>
<li>Número de origem exibido como escritório do Google em Pacifica.
</li>
<li>O golpista alegou que havia uma solicitação de “alteração baseada em certidão de óbito”.
</li>
<li>Enviou depois um e-mail de <code>legal@google.com</code>, assinado por “Norman Zhu”.
</li>
</ul>


</li>

<li>

<p>O truque do código  </p>

<ul>
<li>Em pânico, Scoville leu em voz alta um código de segurança enviado pelo golpista, alegadamente para provar que ainda estava vivo.
</li>
<li>Esse único ato deu ao invasor acesso total à sessão Google e às chaves de 2FA sincronizadas.
</li>
</ul>


</li>

<li>

<p>O estrago  </p>

<ul>
<li>O invasor acessou Gmail, Drive, Fotos e Google Authenticator em nuvem.
</li>
<li>Em 40 minutos, drenou o ETH stakeado e esvaziou a conta Coinbase de Scoville.</li>
</ul>


</li>

</ul>




<h2>
  
  
  Onde o Google deixou brechas
</h2>

<ul>
<li>E-mails falsos parecendo vir de <code>@google.com</code> passaram pelos filtros do Gmail, e no iOS o usuário não conseguiu inspecionar cabeçalhos completos.
</li>
<li>O Authenticator sincronizado por padrão transformou a segunda fator “algo que você possui” em um token recuperável na nuvem  exatamente o que o golpista precisava.</li>
</ul>




<h2>
  
  
  Lições da comunidade
</h2>

<p>Nos comentários do CyNews, desenvolvedores e entusiastas reforçaram:</p>

<ul>
<li>Nunca compartilhe códigos de verificação por telefone ou e-mail.
</li>
<li>Sempre retorne a ligação para um número oficial que você já tenha validado.
</li>
<li>Considere o 2FA sincronizado na nuvem como fator único; adote chaves físicas ou autenticação offline.
</li>
<li>Não atenda chamadas de números desconhecidos — deixe eles deixarem mensagem e confirme depois por canais oficiais.</li>
</ul>




<h2>
  
  
  Boas práticas para se proteger
</h2>

<ol>
<li>Use chaves de hardware (YubiKey, smart cards) para contas críticas.
</li>
<li>Desative a sincronização em nuvem em apps de autenticação, mantenha segredos TOTP localmente.
</li>
<li>Jamais leia códigos de 2FA em voz alta ou encaminhe por telefone/e-mail.
</li>
<li>Em caso de suspeita, desligue e ligue de volta no número oficial do suporte.
</li>
<li>Separe sua conta de e-mail principal das plataformas financeiras e de trabalho.</li>
</ol>




<h2>
  
  
  Considerações finais
</h2>

<p>Até profissionais de segurança podem cair em golpes de engenharia social bem executados. Entender como configurações padrão e limitações de interface podem ser exploradas ajuda a construir defesas mais sólidas. Se esta história fizer alguém hesitar antes de ler um “código urgente” em voz alta, já terá valido a pena compartilhar.</p>




<p>🔗 Substack original: <a href="https://bewildered.substack.com/p/i-was-scammed-out-of-130000-and-google" rel="noopener noreferrer">https://bewildered.substack.com/p/i-was-scammed-out-of-130000-and-google</a><br><br>
📰 Discussão no CyNews: <a href="https://cynews.vercel.app/show/45264726" rel="noopener noreferrer">https://cynews.vercel.app/show/45264726</a></p>

