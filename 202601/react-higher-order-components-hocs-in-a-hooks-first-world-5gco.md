---
Title: React Higher-Order Components (HOCs) in a Hooks-First World
Description: 
Author: Ward Khaddour
Date: 2026-01-04T21:48:51.000Z
Robots: noindex,nofollow
Template: index
---
<p>Sharing logic between React components is no longer about <strong><em>how</em></strong> to reuse logic, but where it belongs — hooks, utilities, render props, and higher-order components all exist — but a question of <strong>where each abstraction actually fits</strong>.</p>

<p>In modern React, hooks dominate the reuse of logic. Yet some problems remain awkward to solve with hooks alone — especially when logic needs to <strong><em>wrap</em></strong>, <strong><em>adapt</em></strong>, or <strong><em>standardize</em></strong> behavior across heterogeneous components, including <strong><em>third-party</em></strong> ones.</p>

<p>In this article, we will examine the use cases, advantages, and disadvantages of HOCs.</p>

<h2>
  
  
  What a Higher-Order Component Really Is
</h2>

<p>Higher-order components provide a clean way to <strong>extend</strong> a component with additional behaviour, without <strong>changing its implementation or ownership</strong>.</p>

<p>The extra logic could be:</p>

<ul>
<li><p><strong>Analytics:</strong> track how many times a button has been clicked.</p></li>
<li><p><strong>Authentication, or authorization gates:</strong> prevent rendering a component based on authentication status or the current user's permissions.</p></li>
<li><p><strong>Third-party library integration:</strong> adapt a third-party component to your application interface.</p></li>
</ul>

<p>At its core, an HOC is a <a href="https://www.geeksforgeeks.org/javascript/pure-functions-in-javascript/" rel="noopener noreferrer"><strong>pure function</strong></a> that takes a component and returns a new component with additional behavior:</p>

<p>Rather than modifying the original component, the HOC <strong>wraps it</strong>, injecting props, behavior, or side effects at the boundary.</p>

<p>HOCs do not change the component itself — they wrap and enhance it.</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const EnhancedComponent = withExtraLogic(BaseComponent);
</code></pre>

</div>

<p>This distinction is important because it defines where responsibility lives in your system.</p>

<blockquote>
<p><strong>By convention</strong>, HOC names start with <code>with</code>.<br>
 While hooks have become popular for logic reuse, HOCs still shine when you want to enhance multiple components without altering their internals.</p>
</blockquote>

<p>HOCs keep logic <em>around</em> a component, while Hooks pull logic <em>into</em> a component.</p>

<h2>
  
  
  HOCs as Design Patterns: Decorator vs Adapter
</h2>

<p>Higher-Order Components are not a design pattern themselves.<br>
They are a <strong>mechanism</strong> that can implement different patterns depending on intent. Recognizing which pattern you are applying helps prevent accidental coupling and misuse of abstraction.</p>

<h3>
  
  
  HOCs as a Decorator
</h3>

<p>An HOC behaves like a <strong>Decorator</strong> when it:</p>

<ul>
<li><p>Preserves the component’s public interface</p></li>
<li><p>Adds behavior without changing how the component is consumed</p></li>
<li><p>Enhances responsibility rather than translating it</p></li>
</ul>

<p>Examples:</p>

<ul>
<li><p>Injecting analytics</p></li>
<li><p>Adding authorization checks</p></li>
<li><p>Enhancing styling or theming</p></li>
<li><p>Adding error boundaries</p></li>
</ul>

<p>In these cases, the wrapped component remains conceptually the same — it is <strong>augmented.</strong></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>withAuth(Component)
withAnalytics(Component)
withTheme(Component)
</code></pre>

</div>

<blockquote>
<p>This aligns closely with the classic Decorator pattern:<br>
<strong>Same interface, more behavior</strong>.</p>
</blockquote>

<h3>
  
  
  HOCs as an Adapter
</h3>

<p>An HOC behaves like an <strong>Adapter</strong> when it:</p>

<ul>
<li><p>Translates between incompatible APIs</p></li>
<li><p>Bridges a component to an external system</p></li>
<li><p>Changes how the component communicates with its environment</p></li>
</ul>

<h2>
  
  
  Exploring an example
</h2>

<p>Forms sit at the center of most front-end applications and are deceptively complex:</p>

<ul>
<li><p>Labels and accessibility wiring</p></li>
<li><p>Validation and error presentation</p></li>
<li><p>Controlled vs uncontrolled inputs</p></li>
<li><p>Integration with non-native inputs</p></li>
</ul>

<p>Libraries like <strong>React Hook Form</strong>, <strong>Formik</strong>, or <strong>TanStack Form</strong> solve state management — but they also introduce <strong>API coupling</strong>.</p>

<p>Consider a signup form built with React Hook Form:</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>'use client'

import { Select } from '@radix-ui/react-select'
import { Controller, useForm } from 'react-hook-form'
import PhoneInput from 'react-phone-number-input'
import 'react-phone-number-input/style.css'

export function SignupForm() {
  const form = useForm({
    defaultValues: {
      name: '',
      email: '',
      phoneNumber: '',
      gender: '',
    },
  })

  const errors = form.formState.errors

  const handleSubmit = form.handleSubmit(console.log)

  return (
    &lt;form onSubmit={handleSubmit}&gt;
      &lt;div className='flex flex-col gap-1'&gt;
        &lt;label htmlFor='name'&gt;Name&lt;/label&gt;
        &lt;input id='name' {...form.register('name')} /&gt;
        {errors.name ? (
          &lt;small className='text-red-500'&gt;{errors.name.message}&lt;/small&gt;
        ) : null}
      &lt;/div&gt;
      &lt;div className='flex flex-col gap-1'&gt;
        &lt;label htmlFor='email'&gt;Email&lt;/label&gt;
        &lt;input id='email' {...form.register('email')} /&gt;
        {errors.email ? (
          &lt;small className='text-red-500'&gt;{errors.email.message}&lt;/small&gt;
        ) : null}
      &lt;/div&gt;
      &lt;div className='flex flex-col gap-1'&gt;
        &lt;label htmlFor='phoneNumber'&gt;Phone Number&lt;/label&gt;
        &lt;Controller
          control={form.control}
          name='phoneNumber'
          render={({ field, fieldState: { error } }) =&gt; (
            &lt;&gt;
              &lt;PhoneInput numberInputProps={{ id: 'phoneNumber' }} {...field} /&gt;
              {error ? (
                &lt;small className='text-red-500'&gt;{error.message}&lt;/small&gt;
              ) : null}
            &lt;/&gt;
          )}
        /&gt;
      &lt;/div&gt;

      &lt;div className='flex flex-col gap-1'&gt;
        &lt;label htmlFor='gender'&gt;Gender&lt;/label&gt;
        &lt;Controller
          control={form.control}
          name='gender'
          render={({ field, fieldState: { error } }) =&gt; (
            &lt;&gt;
              &lt;Select {...field} options={[...]} /&gt;{' '}
              {error ? (
                &lt;small className='text-red-500'&gt;{error.message}&lt;/small&gt;
              ) : null}
            &lt;/&gt;
          )}
        /&gt;
      &lt;/div&gt;

      &lt;button type='submit'&gt;Create Account&lt;/button&gt;
    &lt;/form&gt;
  )
}
</code></pre>

</div>

<p>Notice how every field repeats the same structure:</p>

<ul>
<li><p>A label.</p></li>
<li><p>An input wired to React Hook Form.</p></li>
<li><p>The only thing that changes is which input component is used.</p></li>
</ul>

<p>This is not just a UI problem — it’s a <strong>cross-cutting concern</strong>:</p>

<ul>
<li><p>Field registration, using library-specific APIs (register, Controller)</p></li>
<li><p>Validation feedback</p></li>
<li><p>Accessibility wiring</p></li>
</ul>

<p>This logic is needed by many different components, but it shouldn’t live inside any single one of them.</p>

<p>For a single form, it is okay, but chances are, the app will have a lot of forms, and each will have the same issues.</p>

<p>A common approach is to inject the form library directly into each field component:</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import clsx from 'clsx';

type Props = {
  name: string
  label: string
  control: Control
}

function Input({ name, control, label }: Props) {
  const { field, fieldState } = useController({
    name,
    control,
  })

  const { error } = fieldState

  return (
    &lt;div className='flex flex-col gap-1'&gt;
      &lt;label htmlFor={name} className={'...'}&gt;{label}&lt;/label&gt;
      &lt;input
        {...field}
        className={clsx(
          'border p-2',
          error ? 'border-red-500' : 'border-gray-400'
        )}
    /&gt;
      {error ? (
        &lt;small className='text-red-500'&gt;{error.message}&lt;/small&gt;
      ) : null}
    &lt;/div&gt;
  )
}
</code></pre>

</div>

<p>This works — but it comes with a cost.</p>

<p>Every field component becomes:</p>

<ul>
<li><p>Tightly coupled to React Hook Form</p></li>
<li><p>Hard to reuse outside forms</p></li>
<li><p>Costly to migrate if the form library changes</p></li>
<li>
<p>The same implementation is needed for other types of inputs: Select , PhoneInput, DatePicker</p>

<blockquote>
<p>A UI component (Input) shouldn’t know about a business logic library (RHF)</p>
</blockquote>
</li>
</ul>

<h2>
  
  
  Refactoring with Higher-Order Components
</h2>

<p>Let’s migrate the inputs to</p>

<ul>
<li><p>Reusable input components.</p></li>
<li><p>Isolate library-specific logic.</p></li>
<li>
<p>A single adapter to integrate inputs with libraries.</p>

<blockquote>
<p>The goal is not to reduce code, but to <strong>separate responsibilities cleanly</strong>.</p>
</blockquote>
</li>
</ul>

<h3>
  
  
  Creating a library-agnostic Input component
</h3>

<p>Defining a base input component that knows nothing about any form library</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import clsx from 'clsx';
import { forwardRef } from 'react';

type InputProps = React.InputHTMLAttributes&lt;HTMLInputElement&gt; &amp; {
  error?: { message?: string };
};

export const Input = forwardRef&lt;HTMLInputElement, InputProps&gt;(
  ({ error, ...props }, ref) =&gt; {
    return (
      &lt;input
        {...props}
        ref={ref}
        className={clsx(
          'border p-2',
          error ? 'border-red-500' : 'border-gray-400'
        )}
      /&gt;
    );
  }
);
</code></pre>

</div>

<ul>
<li><p>Simple Input component.</p></li>
<li><p>Accepts standard input props.</p></li>
<li><p>Usable inside or outside a form.</p></li>
<li>
<p>Has no dependency on any form library.</p>

<blockquote>
<p>In React 19, forwardRef is no longer necessary. Pass ref as a prop instead.<br>
forwardRef will be deprecated in a future release. See <a href="https://react.dev/reference/react/forwardRef" rel="noopener noreferrer">forwardRef</a></p>
</blockquote>
</li>
</ul>

<h3>
  
  
  Adapting the Component with a Higher-Order Component
</h3>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import {
  type Control,
  type ControllerRenderProps,
  type FieldError,
  type FieldPath,
  type FieldValues,
  useController,
} from 'react-hook-form';

type WithRhfProps&lt;TFieldValues extends FieldValues&gt; = {
  name: FieldPath&lt;TFieldValues&gt;;
  control: Control&lt;TFieldValues&gt;;
  rules?: object;
};

export function withReactHookForm&lt;TComponentProps = object&gt;(
  Component: React.ComponentType&lt;
    TComponentProps &amp;
      ControllerRenderProps &amp; {
        error?: FieldError;
      }
  &gt;
) {
  const RhfComponent = function &lt;TFieldValues extends FieldValues&gt;(
    props: TComponentProps &amp; WithRhfProps&lt;TFieldValues&gt;
  ) {
    const { name, control, rules, ...rest } = props;

    const {
      field,
      fieldState: { error },
    } = useController({
      name,
      control,
      rules,
    });

    return (
      &lt;div&gt;
        &lt;Component {...(rest as TComponentProps)} {...field} error={error} /&gt;

        {error &amp;&amp; (
          &lt;small className="text-red-500 text-sm"&gt;{error.message}&lt;/small&gt;
        )}
      &lt;/div&gt;
    );
  };

  const displayName = `Rhf(${
    Component.displayName ?? Component.name ?? 'Component'
  })`;

  // Better for Debugging and Readability
  RhfComponent.displayName = displayName;

  return RhfComponent;
}
</code></pre>

</div>

<p>This HOC:</p>

<ul>
<li><p>Encapsulates all React Hook Form logic</p></li>
<li><p>Preserves the original component API</p></li>
<li><p>Acts as an <strong>adapter</strong>, not a replacement. The wrapped component remains portable and unaware of the form system.</p></li>
</ul>

<p>The base component remains isolated from React Hook Form.</p>

<p>While the TypeScript signatures are dense, they are essential for preserving <strong>Prop Transparency</strong> and IDE <strong>autocompletion</strong>.</p>

<p>The way this HOC works is simple — its type notation, after omitting some types for simplicity, is:</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>(Component: React.ComponentType) =&gt; (props) =&gt; JSX.Element
</code></pre>

</div>

<p>The withReactHookForm function accepts a Component and returns a new component. The original component is saved to be used in the extended component, thanks to <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures" rel="noopener noreferrer">Closures</a>.</p>

<h3>
  
  
  Extending The Base Input Without Modifying
</h3>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export const RhfInput = withReactHookForm(Input);
</code></pre>

</div>

<ul>
<li><p>No update is needed for the original component.</p></li>
<li><p>Create a new RhfInput with withReactHookForm</p></li>
<li><p>The Input component is now decoupled from React Hook Form.</p></li>
<li><p>You now have two components, each has its own use case.</p></li>
<li><p>Can wrap native inputs , custom inputs and third-party UI libraries</p></li>
<li><p>Migrating to another form's library is contained in a single adapter layer, create withTanstackForm and it’s done!</p></li>
</ul>

<p>Integration with a third-party library is simple.</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import { withReactHookForm } from './with-react-hook-form';
import TextField from '@mui/material/TextField';

export const RhfTextField = withReactHookForm(TextField);
</code></pre>

</div>

<blockquote>
<p>Notice how form fields now read declaratively, while the integration logic disappears entirely.</p>
</blockquote>

<h3>
  
  
  Usage in Create Account Form
</h3>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>'use client';

import 'react-phone-number-input/style.css';
import { useForm } from 'react-hook-form';
import { RhfInput } from './components/input';
import { RhfSelect } from './components/select';
import { RhfPhoneInput } from './components/phone-input';

export function SignupForm() {
  const form = useForm({
    defaultValues: {
      name: '',
      email: '',
      phoneNumber: '',
      gender: '',
    },
  });

  const handleSubmit = form.handleSubmit(console.log);

  return (
    &lt;form onSubmit={handleSubmit}&gt;
      &lt;div className="flex flex-col gap-1"&gt;
        &lt;label htmlFor="name"&gt;Name&lt;/label&gt;
        &lt;RhfInput control={form.control} name="name" /&gt;
      &lt;/div&gt;
      &lt;div className="flex flex-col gap-1"&gt;
        &lt;label htmlFor="email"&gt;Email&lt;/label&gt;
        &lt;RhfInput control={form.control} name="email" /&gt;
      &lt;/div&gt;
      &lt;div className="flex flex-col gap-1"&gt;
        &lt;label htmlFor="phoneNumber"&gt;Phone Number&lt;/label&gt;
        &lt;RhfPhoneInput control={form.control} name="phoneNumber" /&gt;
      &lt;/div&gt;

      &lt;div className="flex flex-col gap-1"&gt;
        &lt;label htmlFor="gender"&gt;Gender&lt;/label&gt;
        &lt;RhfSelect
          control={form.control}
          name="gender"
          options={['male', 'female', 'other']}
        /&gt;
      &lt;/div&gt;

      &lt;button type="submit"&gt;Create Account&lt;/button&gt;
    &lt;/form&gt;
  );
}
</code></pre>

</div>

<p>The form now reads declaratively, while infrastructure remains hidden.</p>

<p>While state management is now solved, layout concerns (like labels) remain. There are many possible approaches:</p>

<ul>
<li><p>A separate withLabel HOC for uniform layouts</p></li>
<li><p>Render props when the layout varies per form</p></li>
<li>
<p><a href="https://medium.com/@wardkhaddour/building-reusable-ui-in-react-compound-components-render-props-and-api-design-5089b83976e1" rel="noopener noreferrer">Compound components</a> for different layouts and form variations.</p>

<blockquote>
<p>There is no universal solution — what matters is that form state integration is no longer entangled with <strong>layout decisions</strong>.</p>
</blockquote>
</li>
</ul>

<h2>
  
  
  When This Pattern Makes Sense — and When It Doesn’t
</h2>

<p>HOCs might be used when:</p>

<ul>
<li><p>You need to adapt many components to the same infrastructure</p></li>
<li><p>You integrate third-party libraries</p></li>
<li><p>You want to keep the base components portable</p></li>
</ul>

<p>Avoid HOCs when:</p>

<ul>
<li><p>A simple hook suffices</p></li>
<li><p>Component ownership is local</p></li>
<li><p>Abstraction adds more indirection than value</p></li>
<li><p>Debugging stack traces becomes harder</p></li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>Higher-Order Components are no longer the default abstraction in React — but in scenarios involving <strong>cross-cutting concerns</strong>, <strong>third-party integration</strong>, and <strong>long-term maintainability</strong>, HOCs remain a clean, explicit, powerful tool and preserve long-term flexibility in complex applications.</p>

<p>If this article was useful, consider connecting with me on <a href="https://linkedin.com/in/ward-khaddour" rel="noopener noreferrer">LinkedIn</a>.</p>

