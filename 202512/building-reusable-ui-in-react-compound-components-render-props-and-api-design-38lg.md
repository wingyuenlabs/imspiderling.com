---
Title: Building Reusable UI in React: Compound Components, Render Props, and API Design
Description: 
Author: Ward Khaddour
Date: 2025-12-26T22:08:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>Have you ever built a React component that started clean — and a couple of days later had a billion props, conflicting booleans, and a README no one trusts?</p>

<p>Each new boolean doubles the number of possible UI states. By the time you have six booleans, you’re supporting 64 UI variants — most of them undocumented.</p>

<p>Reusable UI is not about writing less code — it’s about designing APIs that <strong>survive change.</strong></p>

<p>Most React components don’t fail because of bugs; they fail because their APIs collapse under real-world requirements.</p>

<p>This article is for developers building shared components, design systems, or complex UI primitives.</p>

<p>We’ll walk through <strong>how and why</strong> to build reusable UI using:</p>

<ul>
<li><p><strong>Compound Components</strong></p></li>
<li><p><strong>Render Props</strong></p></li>
<li><p><strong>A real Accordion implementation</strong></p></li>
<li><p><strong>Trade-offs, alternatives, and improvements</strong></p></li>
</ul>

<h2>
  
  
  The Core Problem: Prop-Driven APIs Don’t Scale
</h2>

<p>Most components start with something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    const [activeItems, setActiveItems] = useState&lt;string[]&gt;([])

    const toggleItem = (id: string)=&gt; { ... }

    &lt;Accordion
      activeItems={activeItems}
      onToggle={toggleItem}
      allowMultiple
      showIcon
      className='...'
      classNames={{
        header:'...',
        body:'...',
        icon:'...',
      }}
    /&gt;
</code></pre>

</div>



<p><strong>At first, it feels fine, then requirements grow:</strong></p>

<ul>
<li><p>Different layouts</p></li>
<li><p>Custom headers</p></li>
<li><p>Conditional behavior</p></li>
<li><p>Design system constraints</p></li>
</ul>

<p><strong>Suddenly:</strong></p>

<ul>
<li>Props explode
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    Accordion
     ├── activeItems
     ├── allowMultiple
     ├── showIcon
     ├── animated
     ├── collapsible
     ├── headerLayout
     └── onToggle
            ↓
       Interdependent logic
       spread across consumers
</code></pre>

</div>



<blockquote>
<p>Each prop looks innocent in isolation, but together they create hidden coupling and undocumented behavior.</p>
</blockquote>

<ul>
<li><p>Usually, the logic is spread across the <em>component’s internals</em>, while the <em>consumer</em> has to manage the state. So the component becomes a “black box” that is hard to tweak.</p></li>
<li><p>Small changes break everything.</p></li>
</ul>

<p><strong>The root issue:</strong> you’re encoding <strong>layout and behavior</strong> into props.</p>

<blockquote>
<p>We’ll refactor this into a compound component by the end.</p>
</blockquote>

<h2>
  
  
  The Mental Model Shift
</h2>

<p>Before writing code, we need the right mental model.</p>

<h3>
  
  
  Old model (Prop-Driven)
</h3>

<blockquote>
<p><em>“I configure a component.”</em></p>
</blockquote>

<ul>
<li><p>Component dictates layout.</p></li>
<li><p>Limited to predefined props.</p></li>
<li><p>Increases complexity with every feature.</p></li>
</ul>

<h3>
  
  
  New model (Composition-Driven)
</h3>

<blockquote>
<p><em>“I assemble behavior from parts.”</em></p>
</blockquote>

<ul>
<li><p>Consumer dictates layout.</p></li>
<li><p>High flexibility with wrapping and reordering elements.</p></li>
<li><p>Complexity stays flat; parts are isolated.</p></li>
</ul>

<p>Instead of telling a component <em>what it should look like,</em> we give it <strong>state and rules</strong>, and let the consumer decide the structure. This is where <strong>Compound Components</strong> and <strong>Render Props</strong> shine.</p>

<blockquote>
<p>This shift changes how you design components — not just how you implement them.</p>
</blockquote>

<h2>
  
  
  What Are Compound Components?
</h2>

<p>Compound Components allow you to expose a <strong>set of related components</strong> that:</p>

<ul>
<li><p>Share state implicitly</p></li>
<li><p>Work only inside a specific parent</p></li>
<li><p>Form a declarative, readable API</p></li>
</ul>

<p>This feels natural because it mirrors how we already think about HTML: <code>select</code> /<code>option</code> and similar HTML pairs, behave like compound components.</p>

<p>Example usage:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    &lt;Accordion&gt;
      &lt;Accordion.Item itemId="one"&gt;
        &lt;Accordion.Header&gt;Title&lt;/Accordion.Header&gt;
        &lt;Accordion.Body&gt;Content&lt;/Accordion.Body&gt;
      &lt;/Accordion.Item&gt;
    &lt;/Accordion&gt;
</code></pre>

</div>



<ul>
<li><p>No prop drilling.</p></li>
<li><p>No configuration hell.</p></li>
<li><p>The structure explains itself.</p></li>
</ul>

<h2>
  
  
  The Mental Model of Compound Components
</h2>

<p>Think of it like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    Accordion (state owner)
     ├── Item (scopes state)
     │    ├── Header (reads + triggers)
     │    └── Body (reads)
</code></pre>

</div>



<blockquote>
<p>State flows down, events flow up — but only within a narrow scope.</p>
</blockquote>

<ul>
<li><p><strong>Accordion</strong> owns the global state</p></li>
<li><p><strong>Item</strong> narrows that state to a single item</p></li>
<li><p><strong>Header / Body</strong> consume only what they need</p></li>
</ul>

<p>Each layer reduces responsibility and knows less — that’s the point, and each component can be understood in isolation.</p>

<p>This is what enables compound components to be maintainable at scale.</p>




<h2>
  
  
  Building the Accordion from Scratch
</h2>

<h3>
  
  
  <strong>1. State Design</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    const [activeItems, setActiveItems] = useState&lt;Set&lt;string&gt;&gt;(...);

    const toggleItem = useCallback(
       (itemId: string) =&gt; {
         if (allowMultiple) {
           setActiveItems(prev =&gt; {
             const newItems = new Set(prev);
             if (newItems.has(itemId)) {
               newItems.delete(itemId);
             } else {
               newItems.add(itemId);
             }
             return newItems;
           });

           return;
         }

         setActiveItems(prev =&gt;
           prev.has(itemId) ? new Set([]) : new Set([itemId])
         );
       },
       [allowMultiple]
     );
</code></pre>

</div>



<blockquote>
<p>The accordion state needs to answer one question efficiently: <em>Is this item open?</em></p>
</blockquote>

<p>Why Set?</p>

<ul>
<li><p>O(1) lookups</p></li>
<li><p>Prevents duplicate IDs automatically</p></li>
<li><p>Natural fit for “multiple open items.”</p></li>
<li><p>Clear semantic intent</p></li>
</ul>

<p>This decision alone enables reuse across multiple UX patterns — and makes it easy to support:</p>

<ul>
<li><p>Both controlled and uncontrolled usage.</p></li>
<li>
<p>Both single and multiple open items.</p>

<blockquote>
<p>This example focuses on uncontrolled usage; a controlled version would accept value and onChange and derive activeItems externally.</p>
</blockquote>
</li>
</ul>

<h3>
  
  
  2. Context as an Internal Contract
</h3>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>type AccordionContextValue = {
  activeItems: Set&lt;string&gt;;
  toggleItem: (id: string) =&gt; void;
};
</code></pre>

</div>

<p>Context is used <strong>internally</strong> — not as a public API. Developers should never need to think about context to use your component.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    Public API
    -------------------
    &lt;Accordion&gt;
      &lt;Accordion.Item /&gt;
    &lt;/Accordion&gt;

    Internal API
    ----------------------
    AccordionContext
      ├── activeItems
      └── toggleItem
</code></pre>

</div>



<blockquote>
<p>Consumers depend on components, not on how those components work internally.</p>
</blockquote>

<p>If consumers depend on your context shape, you’ve leaked your internals.<br>
Compound components enable you to update everything behind the scenes without breaking the consumers.</p>

<p>This is key:</p>

<blockquote>
<p><em>Consumers compose — they don’t manage state.</em></p>
</blockquote>
<h3>
  
  
  3. Creating contexts
</h3>

<p>First, let’s create the contexts<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    import { createContext, useContext } from 'react';

    // Accordion Context
    type AccordionContextValue = {
      allowMultiple?: boolean;
      activeItems: Set&lt;string&gt;;
      toggleItem: (id: string) =&gt; void;
    };

    export const AccordionContext = createContext&lt;AccordionContextValue | null&gt;(
      null
    );

    export const useAccordion = () =&gt; {
      const accordionContext = useContext(AccordionContext);
      if (!accordionContext) {
        throw new Error('useAccordion must be used within &lt;Accordion /&gt;');
      }

      return accordionContext;
    };

    // Accordion Item Context
    type AccordionItemContextValue = {
      id: string;
      isActive: boolean;
    };

    export const AccordionItemContext =
      createContext&lt;AccordionItemContextValue | null&gt;(null);

    export const useAccordionItem = () =&gt; {
      const accordionItemContext = useContext(AccordionItemContext);
      if (!accordionItemContext) {
        throw new Error('useAccordionItem must be used within &lt;AccordionItem /&gt;');
      }

      return accordionItemContext;
    };
</code></pre>

</div>



<p>Using AccordionContext.Provider<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    const getInitialValue = (defaultValue?: string | string[]): string[] =&gt; {
      if (!defaultValue) return [];
      if (Array.isArray(defaultValue)) return defaultValue;

      return [defaultValue];
    };

    function Accordion({ children, allowMultiple, defaultValue }: Props) {
      const [activeItems, setActiveItems] = useState&lt;Set&lt;string&gt;&gt;(
        () =&gt; new Set(getInitialValue(defaultValue))
      );

      const toggleItem = useCallback(
        (itemId: string) =&gt; {
          if (allowMultiple) {
            setActiveItems(prev =&gt; {
              const newItems = new Set(prev);
              if (newItems.has(itemId)) {
                newItems.delete(itemId);
              } else {
                newItems.add(itemId);
              }
              return newItems;
            });

            return;
          }
          setActiveItems(prev =&gt;
            prev.has(itemId) ? new Set([]) : new Set([itemId])
          );
        },
        [allowMultiple]
      );

      const value = useMemo(
        () =&gt; ({ allowMultiple, activeItems, toggleItem }),
        [allowMultiple, activeItems, toggleItem]
      );

      return (
        &lt;AccordionContext.Provider value={value}&gt;
          {children}
        &lt;/AccordionContext.Provider&gt;
      );
    }
</code></pre>

</div>



<h3>
  
  
  4. Scoping with Accordion.Item
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    type ItemProps = {
      children: Children&lt;{ isActive: boolean }&gt;;
      itemId: string;
    };

    function AccordionItem({ children, itemId }: ItemProps) {
      const { activeItems } = useAccordion();

      const isActive = activeItems.has(itemId);

      const { element } = renderChildren(children, { isActive });

      return (
        &lt;AccordionItemContext.Provider value={{ id: itemId, isActive }}&gt;
          {element}
        &lt;/AccordionItemContext.Provider&gt;
      );
    }
</code></pre>

</div>



<p><strong>Why a second context?</strong></p>

<p>Because the <strong>Header and Body should not care about the entire accordion</strong>.<br>
They only care about <em>this</em> item.</p>

<ul>
<li><p>Headers don’t know about other items, and don’t need to.</p></li>
<li><p>Bodies don’t know how toggling works.</p></li>
<li><p>This prevents abstraction leakage.</p></li>
</ul>

<p>Without this layer, every subcomponent would need to understand the <strong>global accordion state</strong>.<br>
That’s how abstractions leak — and complexity spreads.</p>

<blockquote>
<p>This dramatically reduces coupling.</p>
</blockquote>
<h3>
  
  
  5. Render Props — Without Forcing Them
</h3>

<p>Render Props aren’t outdated — they’re just specialized.<br>
They shine when you want escape hatches without committing to new APIs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    &lt;Accordion.Header&gt;
      {({ isActive }) =&gt; &lt;button&gt;{isActive ? 'Open' : 'Closed'}&lt;/button&gt;}
    &lt;/Accordion.Header&gt;
</code></pre>

</div>



<p>And:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    &lt;Accordion.Header&gt;Title&lt;/Accordion.Header&gt;
</code></pre>

</div>



<p>Internally, Accordion.Header checks whether children is a function and calls it if so.</p>

<p>This hybrid approach is powerful because:</p>

<ul>
<li><p>Beginners get clean JSX</p></li>
<li><p>Advanced users get full control</p></li>
<li><p>No API duplication</p></li>
</ul>

<p>How does that work? With a simple utility, we can support Render Prop while preserving normal children.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    export type Children&lt;T&gt; = React.ReactNode | ((props: T) =&gt; React.ReactNode);


    export const renderChildren = &lt;T&gt;(children: Children&lt;T&gt;, props: T) =&gt; {
      if (typeof children === 'function')
        return {
          element: children(props),
          isRenderProp: true,
        };
      return {
        element: children,
        isRenderProp: false,
      };
    };
</code></pre>

</div>



<ul>
<li><p>This avoids <strong>two APIs.</strong></p></li>
<li><p>Consumers opt in to power only when needed.</p></li>
<li><p>Default usage stays clean.</p></li>
</ul>

<p>Usage in Accordion.Header :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    type HeaderProps = {
      children: Children&lt;{ isActive: boolean; onClick: () =&gt; void }&gt;;
      className?: string;
    };

    function AccordionHeader({ children, className }: HeaderProps) {
      const { isActive, id } = useAccordionItem();
      const { toggleItem } = useAccordion();
      const handleToggle = useCallback(() =&gt; toggleItem(id), [id, toggleItem]);
      const { element, isRenderProp } = renderChildren(children, {
        isActive,
        onClick: handleToggle,
      });
      if (isRenderProp) return element;

      return (
        &lt;button
          onClick={handleToggle}
          type="button" 
          aria-expanded={isActive}
          aria-controls={`panel-${id}`}
          id={`button-${id}`}
          className={clsx(
            'flex items-center justify-between w-full cursor-pointer px-3 py-2',
            !isActive &amp;&amp; 'border-b border-gray-400',
            className
          )}
        &gt;
          {element}

          &lt;ChevronDown
            className={clsx('transition-all  h-5 w-5', isActive &amp;&amp; 'rotate-180')}
          /&gt;
        &lt;/button&gt;
      );
    }
</code></pre>

</div>



<p>Note that using Render Prop gives “Total Control” to the user, meaning the user is responsible for the interaction element.</p>

<blockquote>
<p>clsx is a small utility for constructing className strings conditionally.</p>

<p>Consider adding aria- attributes for better accessibility <strong><em>(<a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility" rel="noopener noreferrer">A11y</a>)</em></strong>, and type='button' to prevent form submission.</p>

<p>You should also handle keyboard navigation (Arrow keys, Home/End).</p>
</blockquote>

<ul>
<li><p>We didn’t add a new prop.</p></li>
<li><p>We didn’t branch the API.</p></li>
<li>
<p>We didn’t break existing users.</p>

<blockquote>
<p>Render Props are just specialized.</p>
</blockquote>
</li>
</ul>

<h3>
  
  
  6. The Body: Consuming the State
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    type BodyProps = {
      children: Children&lt;{ isActive: boolean }&gt;;
    };
    function AccordionBody({ children }: BodyProps) {
      const { isActive, id } = useAccordionItem();
      const { element } = renderChildren(children, { isActive });

      return (
        &lt;div
          id={`panel-${id}`}
          aria-labelledby={`button-${id}`}
          className={clsx(
            'overflow-hidden',
            isActive ? 'h-fit border-b border-gray-400' : 'h-0'
          )}
        &gt;
          {element}
        &lt;/div&gt;
      );
    }
</code></pre>

</div>



<blockquote>
<p>The body consumes the isActive state to decide whether to collapse or show content.</p>
</blockquote>

<h3>
  
  
  7. Wrapping it all together
</h3>

<p>After building each component <code>Accordion Root</code>, <code>Item</code>, <code>Header</code>, <code>Body</code> We can export each component alone, or attach them to the Root component and export it alone with simple lines<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    Accordion.Item = AccordionItem;
    Accordion.Header = AccordionHeader;
    Accordion.Body = AccordionBody;

    export { Accordion };
</code></pre>

</div>



<p>The nice thing about this implementation:</p>

<ul>
<li><p>You import a single component and use all its sub-components.</p></li>
<li><p>TypeScript understands it well, and you get full type-safe components.</p></li>
</ul>

<h2>
  
  
  Example Usage
</h2>

<p>So far, we’ve talked about principles and partial snippets.<br>
Let’s ground this in an example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    import { Accordion } from './components/accordion';
    import { FAQs } from './data/faqs';

    function App() {
      return (
        &lt;Accordion allowMultiple&gt;
          {FAQs.map(faq =&gt; (
            &lt;Accordion.Item key={faq.id} itemId={faq.id}&gt;
              &lt;Accordion.Header&gt;{faq.question}&lt;/Accordion.Header&gt;
              &lt;Accordion.Body&gt;{faq.answer}&lt;/Accordion.Body&gt;
            &lt;/Accordion.Item&gt;
          ))}
        &lt;/Accordion&gt;
      );
    }

    export default App;
</code></pre>

</div>



<h3>
  
  
  Another example use-case
</h3>

<ol>
<li>
<strong>Building a Modal component with Open and Close components</strong>
&gt;  These examples illustrate the pattern — not complete implementations.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    function App() {
      return (
        &lt;Modal&gt;
          &lt;Modal.Window name="window-1"&gt;{...}&lt;/Modal.Window&gt;
          &lt;Modal.Open name="window-1"&gt;
            &lt;Button&gt;Open Modal&lt;/Button&gt;
          &lt;/Modal.Open&gt;
          &lt;Modal.Close name="window-1"&gt;
            &lt;Button&gt;Close Modal&lt;/Button&gt;
          &lt;/Modal.Close&gt;
        &lt;/Modal&gt;
      );
    }
</code></pre>

</div>



<p>What’s powerful here is that</p>

<ul>
<li><p>You can render the window in one component and open or close it from anywhere inside the Modal context.</p></li>
<li><p>You can export a hook useModal and control the modal state programmatically.<br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function App() {
  const { openWindow, closeWindow } = useModal();

  const someHandler = () =&gt; {
    // Do some logic

    openWindow('window-1');
   };
}
</code></pre>

</div>



<blockquote>
<p>Notice how the mental model is identical: <strong>One state owner. Many declarative consumers.</strong></p>
</blockquote>

<p><strong>2. Building a File input component with Trigger, Preview, Dropzone, and even Upload Error components</strong></p>

<p>Building a FileInput Component is a nightmare with standard props, especially if you need to include dropzone, preview, and remove files. And it becomes unmaintainable if the preview layout needs to vary across the app.</p>

<p>See this example with <code>Compound Components</code> and <code>Render Prop</code>.</p>

<p>Here we can see how the Render Prop shines.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    function App() {
      return (
        &lt;FileInput&gt;
          &lt;FileInput.Trigger&gt;
            &lt;Button&gt;Upload Image&lt;/Button&gt;
          &lt;/FileInput.Trigger&gt;

          &lt;FileInput.Dropzone&gt;{/**
           * Add a styled dropzone
           * You can add The `FileInput.Trigger` here, it's OK!
           */}
          &lt;/FileInput.Dropzone&gt;

          &lt;FileInput.Preview&gt;
            {(files, removeFile) =&gt; (
              &lt;div&gt;
                {/**
                 * Control how each file is displayed and styled
                 * Add delete button any where you want
                 */}
              &lt;/div&gt;
            )}
          &lt;/FileInput.Preview&gt;
        &lt;/FileInput&gt;
      );
    }
</code></pre>

</div>



<h2>
  
  
  Improving the Accordion: Performance &amp; Architecture
</h2>

<p>At this point, the API is already solid. These improvements are about scale, not correctness, and matter in libraries — not in every app.<br>
Don’t cargo-cult them.</p>
<h3>
  
  
  Improvement 1: Split Contexts for Fewer Re-renders
</h3>

<p>Currently:</p>

<ul>
<li>Any change in activeItems re-renders all consumers</li>
</ul>

<p>We can split contexts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    type AccordionValue = {
      allowMultiple?: boolean;
      activeItems: Set&lt;string&gt;;
    };
    const AccordionStateContext = createContext&lt;AccordionValue | null&gt;(null);


    type AccordionActions = { toggleItem: (id: string) =&gt; void };
    const AccordionActionsContext = createContext&lt;AccordionActions | null&gt;(null);
</code></pre>

</div>



<p>Now:</p>

<ul>
<li><p>This can reduce unnecessary re-renders when combined with memoization.</p></li>
<li><p>Actions can be kept stable.</p></li>
</ul>

<p>These optimizations matter most when your components are widely reused, or if you have 100 accordion items open at once, for example.</p>

<blockquote>
<p><strong>Warning:</strong> This adds complexity — measure before optimizing.</p>
</blockquote>

<h3>
  
  
  Improvement 2: Extract a Headless Hook
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    function useAccordionState({ defaultActive, allowMultiple }) {
      // state + logic only
    }
</code></pre>

</div>



<p>Benefits:</p>

<ul>
<li><p>Testable without UI</p></li>
<li><p>Reusable in different components</p></li>
<li><p>Cleaner separation of concerns</p></li>
</ul>

<p>This turns the Accordion into a <strong>headless + styled hybrid</strong></p>

<h3>
  
  
  Improvement 3: Memoize Subcomponents Strategically
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    const AccordionHeader = memo(function AccordionHeader(...) { ... });
</code></pre>

</div>



<p>Not everywhere — only where re-renders are costly.</p>

<h2>
  
  
  Trade-offs
</h2>

<h3>
  
  
  Pros
</h3>

<ul>
<li><p>Extremely flexible API.</p></li>
<li><p>Clear separation of concerns.</p></li>
<li><p>Scales well in design systems.</p></li>
<li><p>Excellent developer experience.</p></li>
<li><p>Simplifying parts of Accessibility (A11y), because we control the id in the Accordion.Item context, we can automatically link aria-controls and aria-labelledby without the consumer ever having to manually pass IDs.</p></li>
</ul>

<h3>
  
  
  Cons
</h3>

<ul>
<li><p>More code upfront.</p></li>
<li><p>Potential context-based re-render cost.</p></li>
<li><p>Harder debugging than simple prop-based components.</p></li>
<li><p>Not suitable for trivial components.</p></li>
</ul>

<h2>
  
  
  When Should You Use This Pattern?
</h2>

<h3>
  
  
  Use Compound Components when:
</h3>

<ul>
<li><p>Layout varies</p></li>
<li><p>Consumers need composition freedom</p></li>
<li><p>You’re building a shared UI library</p></li>
<li><p>API longevity matters</p></li>
</ul>

<h3>
  
  
  Avoid them when:
</h3>

<ul>
<li><p>The component is simple</p></li>
<li><p>Performance is extremely sensitive</p></li>
<li><p>There’s only one valid layout</p></li>
</ul>

<h2>
  
  
  Limits of Compound Components
</h2>

<p>Compound components are powerful — but they have limits. Understanding them is crucial.</p>

<ol>
<li><strong>Too much freedom can hurt consistency.</strong></li>
</ol>

<p>Consumers can reorder or omit subcomponents, breaking UX or accessibility. Design systems must define <strong>safe composition boundaries</strong>.</p>

<p>Unlike props, requirements (e.g., which subcomponents are required) aren’t explicit. Clear documentation is essential.</p>

<p>Even TypeScript cannot fully enforce:</p>

<ul>
<li><p>Correct composition order</p></li>
<li><p>Required subcomponents</p></li>
<li><p>Semantic correctness</p></li>
</ul>

<p><strong>2. Debugging &amp; performance</strong><br>
Logic is spread across context, hooks, and Render Props, which can make bugs hard to trace. Heavy children or large lists may cause unexpected re-renders.</p>

<p><strong>3. Not every component deserves it</strong></p>

<p>The biggest mistake is <strong>overusing compound components</strong>. They are a poor fit when:</p>

<ul>
<li><p>The layout is fixed</p></li>
<li><p>There’s only one valid structure</p></li>
<li><p>The component is trivial</p></li>
<li><p>The API is unlikely to evolve</p></li>
</ul>

<p>For example:</p>

<ul>
<li><p>Buttons</p></li>
<li><p>Icons</p></li>
<li><p>Badges</p></li>
<li><p>Avatars</p></li>
</ul>

<p>In these cases, compound components add:</p>

<ul>
<li><p>Unnecessary abstraction</p></li>
<li><p>More files</p></li>
<li><p>Harder onboarding</p></li>
</ul>

<p>Avoid them for trivial or fixed-layout components — here, simple props are simpler and safer.</p>

<blockquote>
<p>In short: compound components trade explicit configuration for compositional flexibility. Use deliberately, not by default.</p>
</blockquote>

<h2>
  
  
  Final Thought
</h2>

<blockquote>
<p>Compound Components and Render Props are tools — not goals.</p>
</blockquote>

<p>Reusable UI is not about clever or trendy patterns; it’s about <strong>respecting change</strong>.</p>

<p>The compound components pattern is a very powerful pattern, used in many UI libraries, like:</p>

<ul>
<li><p>Radix UI: see Radix UI <a href="https://www.radix-ui.com/themes/docs/components/tabs" rel="noopener noreferrer">Tabs</a></p></li>
<li><p>MUI: see MUI <a href="https://mui.com/material-ui/react-stepper/" rel="noopener noreferrer">React Stepper</a></p></li>
</ul>

<p>You can find the full Accordion implementation on <a href="https://github.com/WardKhaddour/accordion-compound-component" rel="noopener noreferrer">GitHub</a>.</p>

<p>If this article was useful, consider starring the repo and connecting with me on <a href="https://linkedin.com/in/ward-khaddour" rel="noopener noreferrer">LinkedIn</a>.</p>

