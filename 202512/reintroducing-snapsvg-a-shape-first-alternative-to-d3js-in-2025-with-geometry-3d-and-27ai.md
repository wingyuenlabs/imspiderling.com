---
Title: Reintroducing Snap.svg: A Shape‑First Alternative to D3.js in 2025 (With Geometry, 3D, and TypeScript)
Description: 
Author: Orlin Vakarelov
Date: 2025-12-06T21:48:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>First a quick remark about scope. This is not a full manual for Snap.svg, nor is it a comprehensive comparison with the wider visualization ecosystem. My aim here is narrower and, I hope, more useful: to reintroduce a “classic” SVG library that many people wrote off as abandoned, and to explain why I think it is still a compelling alternative to D3.js for interactive SVG in 2025.</p>

<p>The short version:</p>

<ul>
<li>The original Snap.svg repo at Adobe has been effectively frozen for years.
</li>
<li>Over the last decade I’ve been extending a fork of Snap.svg with geometry utilities, interaction primitives, UI‑building helpers, and TypeScript types, while keeping the original API intact.
</li>
<li>On top of that, there is now a growing toolkit for non‑linear warps, 3D‑ish transforms, and Bezier‑based 3D projections.</li>
</ul>

<p>This post is the first in a series that will unpack those pieces and show how to use them in practice.</p>




<h2>
  
  
  A brief history: Snap.svg and why it mattered
</h2>

<p>Snap.svg was originally created by Dmitry Baranovskiy (of Raphaël fame) and hosted at Adobe as part of their web platform work:</p>

<ul>
<li>Official Adobe repo: <a href="https://github.com/adobe-webplatform/Snap.svg/" rel="noopener noreferrer">https://github.com/adobe-webplatform/Snap.svg/</a>
</li>
</ul>

<p>The core design assumption of Snap.svg was simple but powerful: if the browser already gives you SVG as a first‑class scene graph (paths, groups, transforms, events, etc.), then the JavaScript library should help you work with that directly, not replace it with its own retained‑mode canvas abstraction.</p>

<p>Concretely, this gave you:</p>

<ul>
<li>A thin wrapper around native SVG elements
Snap returns a “paper”; you create shapes, group them, style them, and animate them directly as SVG.</li>
<li>A coherent API for transforms and groupings
You work with groups, transforms, and bounding boxes in a uniform way.</li>
<li>A minimal mental model
You are still working with SVG; Snap just makes that pleasant.</li>
</ul>

<p>That combination made Snap.svg a good fit for icons, dashboards, micro‑interactions, and “SVG as UI” long before “SVG UI” was fashionable.</p>

<p>The official repository, however, has been stagnant for years. No modern builds, no TypeScript, and—crucially—no evolution of the geometry layer. The core ideas are still sound; the ecosystem around them just stopped moving.</p>




<h2>
  
  
  Why Snap.svg (rather than D3.js) in 2025?
</h2>

<p>The obvious question is why you would reach for Snap.svg when D3.js (and a crowd of successors) exists.</p>

<p>Very roughly:</p>

<ul>
<li>
<strong>D3</strong> is data‑first. It shines when you have a non‑trivial data model, need a flexible layout pipeline, and want to bind everything to selections.
</li>
<li>
<strong>Snap.svg</strong> is shape‑first. It shines when you already know the shapes you want (icons, widgets, diagrams, controls) and you want to manipulate them as SVG primitives with a rich geometry API.</li>
</ul>

<p>If what you need is:</p>

<ul>
<li>A draggable diagram editor
</li>
<li>Interactive icons, badges, and widgets
</li>
<li>Geometry‑heavy UIs (flow fields, Voronoi overlays, hulls, 3D‑ish curves)
</li>
<li>SVG‑based UI components that you want to ship as part of a design system
</li>
</ul>

<p>…then the friction of going <em>through</em> D3’s data join model can be more of a distraction than a benefit. At that point, a library that says “give me an <code>&lt;svg&gt;</code>, I will give you a useful geometry/animation/interaction layer” is often closer to what you want.</p>

<p>That is exactly the design stance of this fork.</p>




<h2>
  
  
  The fork: geometry, interaction, UI, and typing on top of Snap.svg
</h2>

<p>The extended fork lives here:</p>

<ul>
<li>
<strong>Forked + extended Snap.svg</strong>: <a href="https://github.com/vakarelov/Snap.svg" rel="noopener noreferrer">https://github.com/vakarelov/Snap.svg</a>
</li>
</ul>

<p>The central constraint is conservative: the original Snap.svg API remains intact. On top of that, the fork adds:</p>

<ul>
<li>
<strong>Bezier / PolyBezier tooling</strong>

<ul>
<li>Higher‑level Bezier helpers (including 3D‑ish curves) and utilities for constructing and editing curves.</li>
</ul>


</li>

<li>

<strong>Polygon operations</strong>

<ul>
<li>Convex and concave hulls, SAT‑style overlap checks, and helpers for morphology‑like operations on polygonal regions.</li>
</ul>


</li>

<li>

<strong>Robust BBox and geometry helpers</strong>

<ul>
<li>Bounding‑box extensions that behave well under transforms, plus richer geometry primitives exposed on elements and the paper.</li>
</ul>


</li>

<li>

<strong>Element and Paper extensions</strong>

<ul>
<li>Higher‑level drawing and layout helpers (for example, grid construction and generalized transforms).</li>
</ul>


</li>

<li>

<strong>Interaction &amp; GUI primitives</strong>

<ul>
<li>“Smart drag” behavior, constrained motion, localized warp fields, and Bezier control‑point editing, all expressed in terms of SVG primitives.</li>
</ul>


</li>

<li>

<strong>Color palettes and UI utilities</strong>

<ul>
<li>Practical helpers for building in‑SVG UI: grids, overlays, control panels, labels, etc.</li>
</ul>


</li>

<li>

<strong>Comprehensive TypeScript types</strong>

<ul>
<li>A proper type layer for the Snap API and the new geometry/interaction extensions, which makes working in modern tooling substantially more tractable.</li>
</ul>


</li>

</ul>

<p>The point is not to turn Snap into something else; it is to make the “shape‑first” approach viable for the kinds of problems front‑end developers actually face now.</p>




<h2>
  
  
  Towards 3D and non‑linear transforms
</h2>

<p>So far I have spoken mostly in the abstract: geometry, interaction, “UI helpers”. To make this more concrete, it is useful to look at the new <strong>non‑linear transform</strong> and <strong>3D‑ish</strong> tools, because they stress‑test everything else.</p>

<h3>
  
  
  Non‑linear transform gallery
</h3>

<p>The non‑linear demo page wires Snap into a set of animated “fields” that bend a grid in time:</p>

<ul>
<li>Some presets sweep a sine wave along a diagonal axis.
</li>
<li>Others behave like a cantilever, with the grid “hinging” around a line.
</li>
<li>There are radial ripples, bulges, and compound warps combining several effects.</li>
</ul>

<p>You can see them here:</p>

<ul>
<li>Non‑linear transforms:
<a href="https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/non-linear/index.html" rel="noopener noreferrer">https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/non-linear/index.html</a>
</li>
</ul>

<p>Conceptually, each demo defines a time‑dependent transform over the SVG plane and asks Snap to apply it to every point of a grid. Many of these fields internally use 3D transforms (rotations in depth, perspective projection) and then project back to 2D, which is why they feel “3D‑ish” even though they are rendered in plain SVG.</p>

<p>The point is not that every UI will use such warps; it is that the extension layer can describe genuine fields over the SVG plane, rather than only global affine transforms.</p>

<h3>
  
  
  Localized warps and “warp fields”
</h3>

<p>The warp demo takes the same machinery but makes it local and interactive:</p>

<ul>
<li>A circular “warp field” is drawn on the stage.
</li>
<li>A smaller “probe” grid can be dragged around inside that field.
</li>
<li>Depending on the preset (twist/pinch, bulge, ripple, 3D sine depth), a localized warp is applied only where the probe intersects the field.</li>
</ul>

<p>You can try it here:</p>

<ul>
<li>Warp fields (localized non‑linear warps):
<a href="https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/warp/warp_demo.html" rel="noopener noreferrer">https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/warp/warp_demo.html</a>
</li>
</ul>

<p>This is the sort of interaction Snap.svg is particularly suited for: SVG is your scene graph, and the library gives you a programmable metric on that graph. Warp fields are defined declaratively; dragging and bounding‑box helpers do the bookkeeping.</p>

<h3>
  
  
  3D Bezier projection demo
</h3>

<p>Non‑linear warps are one axis of extension; the other is <strong>explicit 3D curves and projections</strong>. The Bezier projection demo illustrates this:</p>

<ul>
<li>You control a 3D Bezier curve via sliders for the Z‑coordinates and direct manipulation of control points in multiple orthographic views.
</li>
<li>The same curve is shown simultaneously in:

<ul>
<li>XY, XZ, and YZ orthographic projections
</li>
<li>A perspective camera view
</li>
<li>A custom “isometric‑like” projection</li>
</ul>


</li>

</ul>

<p>The demo is here:</p>

<ul>
<li>3D Bezier projections:
<a href="https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master//demos/bezier/bezier_projection_demo.html" rel="noopener noreferrer">https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master//demos/bezier/bezier_projection_demo.html</a>
</li>
</ul>

<p>Internally, the curve is represented by an extended Bezier object that supports 3D control points and several projection methods. You can also animate the camera around the curve, orbiting in the XZ plane while maintaining a fixed target.</p>

<p>This is a step towards a <strong>full SVG‑based 3D engine for Snap.svg</strong>: the geometry layer has to be coherent enough that 3D curves, projections, and camera motion feel like natural extensions of existing 2D concepts (paths, bounding boxes, transforms), not like an unrelated subsystem bolted on the side.</p>




<h2>
  
  
  Other demos you can click through right now
</h2>

<p>If you prefer to start from behavior rather than abstractions, the easiest way into the fork is to explore the demos. They are all built directly on the extended Snap API (no hidden frameworks).</p>

<ul>
<li><p><strong>Easing functions in mina.js</strong><br><br>
Many additional easing functions plus a small “easing construction engine”, with a visual explorer wired to <code>mina</code> (Snap’s animation engine), to help you build temporal curves that match the spatial smoothness of the new geometry tools.<br><br>
<a href="https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/easings/index.html" rel="noopener noreferrer">https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/easings/index.html</a></p></li>
<li><p><strong>Point‑utils: Voronoi</strong><br><br>
Demonstrates the point‑geometry utilities by computing a Voronoi diagram over point sets and rendering the cells as interactive SVG polygons.<br><br>
<a href="https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/point-utils/voronoi/index.html" rel="noopener noreferrer">https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/point-utils/voronoi/index.html</a></p></li>
<li><p><strong>Point‑utils: Proximity</strong><br><br>
A companion demo for proximity and neighborhood queries on point sets, again rendered directly in SVG.<br><br>
<a href="https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/point-utils/proximity/index.html" rel="noopener noreferrer">https://htmlpreview.github.io/?https://github.com/vakarelov/Snap.svg/master/demos/point-utils/proximity/index.html</a></p></li>
</ul>

<p>Each of these is deliberately written as “just” Snap code: no bundler is required to understand the core ideas, and (once you build the fork) you can view the sources alongside the running demos.</p>




<h2>
  
  
  Getting started with the fork
</h2>

<p>The GitHub repository of the extended Snap.svg lives here:</p>

<ul>
<li><a href="https://github.com/vakarelov/Snap.svg" rel="noopener noreferrer">https://github.com/vakarelov/Snap.svg</a></li>
</ul>

<p>A quick path to experimentation is:</p>

<ol>
<li>Clone the repo.
</li>
<li>Build the distribution bundle.
</li>
<li>Include the built file in a static HTML page, in the same spirit as the original Snap.svg library.
</li>
<li>Attach Snap to an existing <code>&lt;svg&gt;</code> and start drawing/manipulating shapes using the extended APIs (geometry helpers, warps, Bezier tools, point utilities, etc.), borrowing patterns from the demo sources.</li>
</ol>

<p>If you are already using Snap.svg from the original Adobe repo, the existing API should continue to work; the new pieces are additive.</p>




<h2>
  
  
  What comes next in this series
</h2>

<p>This post has been mostly conceptual: where Snap.svg comes from, why I think it still deserves attention, and how the fork extends it towards a richer geometry and 3D story.</p>

<p>In the next posts I plan to:</p>

<ul>
<li>Walk through the <strong>geometry primitives</strong> in more detail (Bezier/PolyBezier, polygon operations, hulls, SAT overlap).
</li>
<li>Show how to build <strong>interactive widgets</strong> with the drag helpers, BBox utilities, and warp fields.
</li>
<li>Dive deeper into the <strong>3D Bezier and camera model</strong>, and eventually into the full SVG‑based 3D engine that is being built on top of these pieces.</li>
</ul>

<p>Questions or gaps? Open an issue in the GitHub repo or ping in the relevant PR—I'm happy to add examples or fill in missing documentation as we go.</p>

<p>This is, in that sense, a reintroduction rather than a eulogy: Snap.svg is not a frozen artifact of the early 2010s, but a living tool that still has quite a bit of headroom for modern interactive SVG work.</p>

