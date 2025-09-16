---
Title: Recriando Smartphones: DIY de Flip Phone com Case CNC e Teclado Físico
Description: 
Author: Mr Punk da Silva
Date: 2025-09-16T21:44:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>Um experimento do maker Marcin Plaza chegou ao CyNews e viralizou nas redes: reaproveitar telas dobráveis de Galaxy Z Flip quebrados para criar um flip phone totalmente funcional, usinando um case em alumínio e integrando um teclado estilo BlackBerry. No vídeo “I built my own Phone… because innovation is sad rn” (2,3 M visualizações), ele não só mostra o processo passo a passo, mas discute conceitos como reparabilidade, modularidade e design de hardware — temas que geraram debates intensos no fórum do CyNews.</p>

<p>📰 Cobertura no CyNews: <a href="https://cynews.vercel.app/show/45245050" rel="noopener noreferrer">https://cynews.vercel.app/show/45245050</a><br><br>
🎥 Vídeo completo: <a href="https://youtu.be/qy_9w_c2ub0" rel="noopener noreferrer">https://youtu.be/qy_9w_c2ub0</a>  </p>




<h2>
  
  
  Visão Geral do Projeto
</h2>

<p>Marcin parte de aparelhos danificados, aproveita apenas os componentes essenciais e reconstrói:</p>

<ul>
<li>extração da tela dobrável e dobradiças de dois Galaxy Z Flip
</li>
<li>modelagem CAD de um case em duas metades de alumínio
</li>
<li>mecanismo de quatro barras que recria o efeito flip
</li>
<li>integração de um teclado físico BlackBerry via Arduino Pro Micro (USB HID)
</li>
<li>adaptação da placa controladora de tela e circuitos de alimentação
</li>
</ul>

<p>O firmware roda um Android AOSP customizado, permitindo ligações, SMS e uso básico de apps. O teclado físico é detectado como um dispositivo de entrada padrão, devolvendo a experiência tátil dos antigos celulares.</p>




<h2>
  
  
  Conceitos Importantes
</h2>

<ol>
<li>reparabilidade e direito a conserto
fabricantes limitam acesso a peças e documentações; aqui, tudo é aberto
</li>
<li>design modular
cada bloco funcional (tela, teclado, bateria) pode ser substituído isoladamente
</li>
<li>upcycling e sustentabilidade
prolonga a vida útil de componentes caros, reduzindo descarte eletrônico
</li>
<li>mecânica aplicada
uso de quatro barras para obter o movimento de flip, inspirado em engenharia automobilística
</li>
<li>interface humano-computador
estudo da ergonomia e sensação tátil no uso do teclado físico
</li>
</ol>




<h2>
  
  
  Detalhes Técnicos do Vídeo
</h2>

<ul>
<li>CAD e CNC
Marcin compartilha arquivos STEP e G-code que servem de base para fresagem em 5 eixos ou, com adaptações, em máquinas hobby de 3 eixos
</li>
<li>Arduino Pro Micro como controlador de teclado
mapeamento de cada tecla BlackBerry para HID USB, gerando eventos padrão no Android
</li>
<li>Novos conectores
pinos SMT personalizados e adaptadores 3D-print para fixar cabos FFC da tela
</li>
<li>Firmware Android
uma build AOSP minimalista, com drivers externos carregados via init.d
</li>
</ul>

<p>O resultado une design industrial e experimentação de software livre, mostrando que qualquer maker com acesso a CNC e um pouco de eletrônica pode replicar a montagem.</p>




<h2>
  
  
  Reações da Comunidade (CyNews)
</h2>

<ul>
<li>“seria incrível ver isso como um kit open-source para hackerspaces”
</li>
<li>“quero os arquivos do case; quanto tempo total de usinagem?”
</li>
<li>“como ficou a autonomia da bateria? vale a pena trocar?”
</li>
<li>“movimento four-bar é clássico em suspensão veicular; boa adaptação”
</li>
<li>“isso redefine o que significa custom phone em 2025”
</li>
</ul>

<p>Muitos elogiaram a filosofia de devolver ao usuário o controle sobre o próprio hardware — posicionando o projeto no epicentro do movimento right-to-repair.</p>




<h2>
  
  
  Por Que Esse Projeto Importa
</h2>

<ul>
<li>demonstra o poder do hardware open-source e da cultura maker
</li>
<li>explora integrações inusitadas entre eletrônica, mecânica e software
</li>
<li>inspira discussões sobre sustentabilidade e design centrado no usuário
</li>
<li>empodera entusiastas a reutilizarem componentes caros de forma criativa
</li>
</ul>

<p>Se você curte desmontar gadgets, fresagem CNC e mapeamento USB, este case de flip phone é um convite para repensar nossos dispositivos do dia a dia.</p>




<p>🔗 Projeto no PCBWay (arquivos e detalhes): <a href="https://www.pcbway.com/project/shareproject/I-Built-My-Own-Phone" rel="noopener noreferrer">https://www.pcbway.com/project/shareproject/I-Built-My-Own-Phone</a><br><br>
📰 Leia a cobertura no CyNews: <a href="https://cynews.vercel.app/show/45245050" rel="noopener noreferrer">https://cynews.vercel.app/show/45245050</a></p>

