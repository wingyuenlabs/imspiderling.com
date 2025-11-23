---
Title: Como foi palestrar na Clojure South 2025
Description: 
Author: Marcio Frayze
Date: 2025-11-23T21:40:28.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  O que é a Clojure South?
</h2>

<p>Segundo <a href="http://clojure-south.com" rel="noopener noreferrer">site oficial</a>:</p>

<blockquote>
<p>"Organizada pelo Nubank, a Clojure South faz parte da programação oficial da comunidade de Clojure, conectando pessoas desenvolvedoras, entusiastas e empresas para compartilhar experiências reais, discutir tendências e fortalecer a rede global da linguagem."</p>
</blockquote>

<h2>
  
  
  Motivação
</h2>

<p>Em uma conversa com uma brasileira que conheci na <a href="https://www.lambdadays.org/lambdadays2025" rel="noopener noreferrer">Lambda Days 2025</a> (falei sobre como foi participar deste evento <a href="https://dev.to/marciofrayze/como-foi-minha-experiencia-na-lambda-days-2025-27k3">neste artigo</a>), perguntei se ela iria assistir às palestras (ela trabalha para a Nubank) e descobri que ela iria ser instrutora do <em>workshop</em> de Clojure! Nesta conversa ela sugeriu que eu deveria enviar uma proposta de palestra. Comentei uma ideia que eu tinha para submeter, mas que estava meio inseguro de enviar. Ela me encorajou a tentar e eu resolvi arriscar!</p>

<p>Embora tenha no meu currículo <a href="(https://segunda.tech/sobre)">algumas palestras</a> e <a href="https://segunda.tech/tags/podcast/" rel="noopener noreferrer">participações em podcasts</a>, sentia que talvez fosse mais interessante, neste caso, submeter uma proposta no formato <em>lighting talk</em> (palestra de apenas 10 minutos e sem tempo para perguntas).</p>

<p>E algum tempo depois, para minha surpresa, a proposta foi aceita!</p>

<h2>
  
  
  Tema
</h2>

<p>Havia pensado em enviar propostas sobre dois possíveis temas. Um sobre o ensino de Clojure e Programação Funcional para iniciantes (por conta do meu curso <a href="https://segunda.tech/clojure" rel="noopener noreferrer">Clojure: Introdução à Programação Funcional</a>). E outro sobre um projeto que fiz na empresa onde trabalho, que utiliza Clojure no <em>backend</em> e a linguagem de programação <a href="https://elm-lang.org" rel="noopener noreferrer">Elm</a> para o <em>front-end</em>.</p>

<p>Optei pela segunda opção. Já era um projeto que eu queria falar sobre publicamente e esta parecia ser uma boa oportunidade! </p>

<p>Outro fator é que o projeto em questão está relacionado com um <em>App</em> bastante famoso no Brasil, a Carteira Digital de Trânsito — <em>App</em> desenvolvido pelo <a href="https://www.serpro.gov.br" rel="noopener noreferrer">SERPRO</a> e disponível para <a href="https://play.google.com/store/apps/details?id=br.gov.serpro.cnhe" rel="noopener noreferrer">Android</a> e <a href="https://apps.apple.com/br/app/carteira-digital-de-tr%C3%A2nsito/id1275057217" rel="noopener noreferrer">iOS</a>. Imaginava que isso poderia aumentar as chances de a palestra ser aceita.</p>

<p>A descrição completa que enviei foi:</p>

<blockquote>
<p>Durante o desenvolvimento da <em>Carteira Digital de Trânsito</em>, percebi como era desafiador dar suporte a um <em>App</em> com milhões de usuários integrado a diversos sistemas. Sonhei em criar uma interface interna simples e amigável para consultas que facilitassem o atendimento. Assim surgiu o <strong>Apoio CDT</strong>, um sistema web que automatizou processos e permitiu consultas rápidas e seguras. Com <em>backend</em> em Clojure e <em>front-end</em> em Elm, desenvolvemos um MVP em três semanas. O resultado foi uma arquitetura web simples, eficiente e fácil de manter ao longo dos anos.</p>
</blockquote>

<p>Até onde sei, este foi o primeiro projeto da empresa que utilizada Clojure e também o primeiro a utilizar Elm. E esta foi a primeira vez que tive oportunidade de falar sobres este sistema para pessoas de fora da empresa onde trabalho!</p>

<h2>
  
  
  Demais palestras e workshops
</h2>

<p>O evento durou dois dias, sendo que no primeiro ocorreram os 2 <em>workshops</em>:</p>

<h3>
  
  
  <em>Workshop</em> de Clojure
</h3>

<p>Um curso de introdução ao Clojure, em inglês, ministrado pelo <a href="https://www.linkedin.com/in/christoph-neumann-6089438/" rel="noopener noreferrer">Christoph Neumann</a>, <em>Developer Advocate</em> de Clojure no Nubank.</p>

<p>Embora eu já tenha experiência com Clojure (e inclusive seja instrutor do curso <a href="https://www.udemy.com/course/clojure-introducao-a-programacao-funcional/" rel="noopener noreferrer">Clojure: Introdução à Programação Funcional</a>), foi muito legal reforçar os principais conceitos desta linguagem e também aprendi alguns fundamentos que ainda não conhecia!</p>

<h3>
  
  
  <em>Workshop</em> de Datomic
</h3>

<p>O período da tarde do primeiro dia foi destinado ao <em>workshow</em> do banco de dados <a href="https://www.datomic.com" rel="noopener noreferrer">Datomic</a>. Este foi ministrado em português pela <a href="https://www.linkedin.com/in/aanacarolina/" rel="noopener noreferrer">Carolina Silva</a> (a mesma que citei no começo deste artigo) e <a href="https://www.linkedin.com/in/hanna-f-mariano-59578596/" rel="noopener noreferrer">Hanna Figueiredo</a>, ambas <em>Software Engineering</em> no Nubank.</p>

<p>Eu já havia assistido várias palestras sobre o Datomic e sempre achei um banco de dados muito intrigante e interessante. Acredito que faria muito sentido utilizá-lo em vários dos projetos que atuo ou já atuei no passado.</p>

<p>O <em>workshop</em> mostrou vários conceitos teóricos do Datomic, mas também foi muito prático.</p>

<p>Você pode conferir a parte prática do <em>workshop</em> <a href="https://github.com/Datomic/day-of-datomic-conj" rel="noopener noreferrer">neste repositório do GitHub</a>!</p>

<p>Confesso que foi muitas informações em pouco tempo. Definitivamente, vou precisar voltar ao repositório e praticar bastante ainda para internalizar os principais conceitos deste banco de dados. Mas valeu muito a pena!</p>

<p>Se antes eu achava que valeria a pena estudar este banco, agora tenho certeza!</p>

<h2>
  
  
  Conversas e <em>feedbacks</em>
</h2>

<p>Logo ao chegar no evento tive a oportunidade de conversar com algumas pessoas que só conhecia virtualmente, como o <a href="https://www.linkedin.com/in/arthur-fucher/" rel="noopener noreferrer">Arthur Fücher</a>. Além de ser <em>Senior Software Engineer</em> no Nubank, Arthur é uma figura muito presente na comunidade Clojure do Brasil!</p>

<p>E mesmo sendo uma pessoa bastante tímida, foi possível conhecer muitas pessoas bacanas do Brasil e de outros países.</p>

<p>Em especial, fui <strong>muito</strong> bem recebido pelo Christoph Neumann. Eu estava parado em um canto no <em>happy hour</em>, após o segundo dia de evento, comendo e pensando em já ir pra casa, quando ele se aproximou e puxou conversa. Conversamos um pouco e comecei a fazer algumas perguntas sobre Datomic. Ao longo da conversa, citei que talvez fosse um banco de dados interessante para empresa onde trabalho. Quando falei onde trabalhava e o que a empresa fazia, ele se mostrou bastante interessado! E logo se ofereceu para me apresentar o <a href="https://www.linkedin.com/in/joe-lane-7674173a/" rel="noopener noreferrer">Joe Lane</a>, <em>Principal Engineer Building Datomic</em> na Nubank!</p>

<p>Conversei com o Joe e, mais uma vez, fui super bem recebido! Ele mostrou bastante entusiasmo quando comentei onde trabalhava e os tipos de projetos que o SERPRO desenvolve, e ficou à disposição para tirar dúvidas sobre Datomic, inclusive comentando que o SERPRO poderia ser um ótimo <em>case</em> para o Datomic.</p>

<p>Ao longo do evento também tive a oportunidade de conhecer algumas pessoas que fizeram <a href="https://www.udemy.com/course/clojure-introducao-a-programacao-funcional/" rel="noopener noreferrer">meu curso de Clojure</a>! Entre essas pessoas, conheci o <a href="https://www.linkedin.com/in/bruno-dobelin/" rel="noopener noreferrer">Bruno Guimarães</a>, que atualmente é <em>Senior Software Engineer</em> no Nubank e foi <em>host</em> da conferência.</p>

<p>Conhecer pessoalmente pessoas que fizeram meu curso online é sempre uma experiência muito legal! Os números, as estatísticas, os comentários, ganham um cara, uma voz, vida. É sempre muito gratificante!</p>

<p>No segundo dia, após a minha apresentação, muitas pessoas vieram conversar e querer saber mais sobre como consegui introduzir Clojure em um sistema do governo federal brasileiro. Tiveram algumas dúvidas mais pontuais, sobre minha motivação para usar <a href="https://elm-lang.org" rel="noopener noreferrer">Elm</a> no <em>front-end</em> (e não optar por utilizar Clojure em sua versão <a href="https://clojurescript.org" rel="noopener noreferrer">ClojureScript</a>), entre várias outras conversas interessantes.</p>

<p>O <em>feedback</em> foi bastante positivo e todas as conversas foram muito bacanas, bem humoradas e respeitosas. </p>

<h2>
  
  
  Comidas (e mais conversas)
</h2>

<p>O evento contava com <em>welcome coffee</em> no início do dia e <em>coffee breaks</em> ao longo do dia. A qualidade era muito boa e tudo muito formal (talvez formal até um pouco demais? rs).</p>

<p>Já para almoçar, existiam várias opções (não inclusas no valor do ingresso) de restaurantes dentro do mesmo complexo de prédios onde encontra-se a <a href="https://building.nubank.com/pt-br/escritorio-nubank-brasil-spark/" rel="noopener noreferrer">Nubank Sparks</a>. Isso facilitava bastante, já que não precisava sair do local, especialmente no primeiro dia, de workshops, já que estava com meu notebook.</p>

<p>Você pode almoçar onde quiser. Mas gostei das opções que estavam disponíveis ali perto e foi mais uma oportunidade de conversar com mais pessoas do evento. Novamente, mesmo sendo bastante tímido e sentando em um lugar mais afastado, logo vieram várias pessoas sentar junto comigo e puxar conversa. Neste momento conheci uma Ucraniana que mostrou pra gente o App <em>Diia</em>, onde ela pode ter versões digitais de vários de seus documentos, como Carteira de Motorista, Passaporte, entre outros e ter acesso a mais de 130 serviços do governo Ucraniano (você pode saber mais sobre este App <a href="https://en.wikipedia.org/wiki/Diia" rel="noopener noreferrer">nesta página do Wikipedia</a>)!</p>

<h2>
  
  
  Valeu a pena?
</h2>

<p>Definitivamente, sim!</p>

<p>O evento ocorreu em dois dias da semana (uma segunda e terça-feira). Isso talvez atrapalhe a participação de algumas pessoas — um colega meu não conseguiu participar pois a empresa não liberou este dias para ele, por exemplo. Mas para aquelas pessoas que conseguem participar, fica muito menos cansativo do que ter um evento como este no final de semana, depois de uma longa semana de trabalho.</p>

<p>Eu havia comprado os ingressos na pré-venda, antes de ter minha palestra aprovada, mas depois tive o valor estornado pela coordenação do evento - ganhei ingressos para os dois dias por ser palestrante. Também me ofereceram ajuda para transporte e estadia, a qual eu recusei, já que o evento seria próximo da minha residência.</p>

<p>A experiência de palestrar em um evento desta magnitude, mesmo que por apenas 10 minutos, foi incrível! É uma sensação de muita gratidão às pessoas que ajudaram a organizar a conferência e que confiaram no meu potencial!</p>

