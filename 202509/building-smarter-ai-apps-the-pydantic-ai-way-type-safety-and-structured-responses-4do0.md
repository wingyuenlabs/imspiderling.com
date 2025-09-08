---
Title: Building Smarter AI Apps: The Pydantic AI Way (Type Safety and Structured Responses)
Description: 
Author: Gabriel Melendez
Date: 2025-09-08T21:15:12.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0o7kq3gk9xopkjq4j4d6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0o7kq3gk9xopkjq4j4d6.png" alt="Building Smarter AI Apps: The Pydantic AI Way (Type Safety and Structured Responses)" width="800" height="533"></a></p>

<p>The landscape of AI Development has been transformed by large language models, but building production-ready AI applications remains challenging. Developers often struggle with unpredictable outputs, inconsistent data formats, and the complexity of integrating AI models into existing Python codebases. Pydantic AI is a useful agent framework that solves these pain points by bringing the reliability and developer experience of modern Python libraries to Gen AI.</p>

<h3>
  
  
  What is Pydantic AI and Why Does It Matter?
</h3>

<p>Designed to make it less painful to build production grade applications with Generative AI.</p>

<p>Traditional AI development often feels like this:</p>

<ul>
<li>You send a prompt to an LLM</li>
<li>You get back a string response that might be formatted correctly or maybe not</li>
<li>You spend hours writing custom parsing logic</li>
<li>You handle edge cases, the AI returns unexpected formats</li>
<li>Your application breaks in production when AI decides to be creative with its responses</li>
</ul>

<p>Pydantic AI eliminates this chaos by ensuring AI responses conform to predefined, validated data structures. Instead of wrestling with unpredictable string responses, you get typed Python objects that integrate seamlessly with your existing codebase.</p>

<p>Now let's see how to implement Pydantic AI with a few examples.</p>

<ol>
<li>Type Safety and Validation First</li>
</ol>

<p>When you define a response structure, Pydantic AI ensures the AI's output always conforms to that structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from pydantic import BaseModel
from pydantic_ai import Agent

# Define exactly what you expect back
class WeatherResponse(BaseModel):
    temperature: int
    condition: str
    humidity: float
    forecast_confidence: str # "high", "medium", "low"

# Create an agent that MUST return this structure
weather_agent = Agent('openai:gpt-4o', result_type=WeatherResponse)

# This will ALWAYS return a WeatherResponse object, never a string
result = await weather_agent.run("What's the weather in New York?")
print(f"Temperature: {result.data.temperature}°F") # Type-safe access

</code></pre>

</div>



<ol>
<li>Structured Responses with Built-in Validation</li>
</ol>

<p>The output will be validated with Pydantic to guarantee it is a SupportOutput, since the agent is generic, it'll also be typed as a SupportOutput to aid with static type checking. This means:</p>

<ul>
<li>No more parsing failures</li>
<li>Consistent data formats</li>
<li>Immediate error detection</li>
</ul>

<ol>
<li>Python-Centric Design Philosophy</li>
</ol>

<p>Pydantic AI leverages standard Python practices you already know.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from typing import List, Optional
from enum import Enum

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium" 
    HIGH = "high"

class Task(BaseModel):
    title: str
    description: str
    priority: Priority
    due_date: Optional[str] = None
    tags: List[str] = []

</code></pre>

</div>



<p>If you understand Pydantic models, you understand Pydantic AI responses</p>

<ol>
<li>Built-in Error Handling and Retries</li>
</ol>

<p>When validations fails, Pydantic AI doesn't just crash, retries by sending the validation error back to the model with instructions to fix the response<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from pydantic_ai import ModelRetry

@agent.tool
def validate_email(email: str) -&gt; str:
    """Validate an email address format."""
    if "@" not in email or "." not in email.split("@")[1]:
        raise ModelRetry("Please provide a valid email address with @ and domain")
    return email

</code></pre>

</div>



<h3>
  
  
  Getting Started: Step-by-Step Implementation of Pydantic AI
</h3>

<p>Let's build a comprehensive understanding through progressively complex samples.</p>

<p><strong>Step 1: Basic Agent Setup</strong></p>

<p>First, install Pydantic AI and set your environment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>pip install pydantic-ai
export OPENAI_API_KEY='your-api-key-here'

</code></pre>

</div>



<p>Create your first structured AI responses:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from pydantic import BaseModel
from pydantic_ai import Agent
import asyncio

# Step 1: Define your response structure
class BookRecommendation(BaseModel):
    title: str
    author: str
    genre: str
    rating: float # 1.0 to 5.0
    reason: str
    similar_books: list[str] = []

# Step 2: Create an agent with this response structure
book_agent = Agent(
    'openai:gpt-4o',
    result_type=BookRecommendation,
    system_prompt="""You are a knowledgeable librarian. When recommending books, 
    always provide ratings between 1.0 and 5.0, and suggest 2-3 similar books."""
)

# Step 3: Use the agent
async def get_recommendation():
    result = await book_agent.run("I love mystery novels with complex characters")
    book = result.data # This is guaranteed to be a BookRecommendation object

    print(f"📚 {book.title} by {book.author}")
    print(f"⭐ Rating: {book.rating}/5.0")
    print(f"📝 Why: {book.reason}")
    print(f"📖 Similar books: {', '.join(book.similar_books)}")

# Run the example
asyncio.run(get_recommendation())

</code></pre>

</div>



<p><strong>Step 2: Adding Function Tools for External Integration</strong></p>

<p>Function tools allow your AI agents to interact with external systems and APIs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from pydantic_ai import Agent
from datetime import datetime
import httpx

# Create an agent that can access external functions
weather_agent = Agent('openai:gpt-4o')

@weather_agent.tool
async def get_current_weather(city: str) -&gt; dict:
    """Get the current weather for a specific city using a weather API."""
    # Simulated API call (in production, use a real weather API)
    weather_data = {
        "temperature": 72,
        "condition": "sunny",
        "humidity": 45,
        "wind_speed": 8
    }
    return weather_data

@weather_agent.tool  
async def get_current_time() -&gt; str:
    """Get the current date and time."""
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

@weather_agent.tool
async def calculate_comfort_index(temperature: int, humidity: int) -&gt; str:
    """Calculate a comfort index based on temperature and humidity."""
    if temperature &lt; 60 or temperature &gt; 80:
        return "uncomfortable"
    elif humidity &gt; 60:
        return "muggy"
    else:
        return "comfortable"

# Use the agent with access to all these tools
async def weather_demo():
    result = await weather_agent.run(
        "What's the weather like in San Francisco and how comfortable is it?"
    )
    print(result.data)

</code></pre>

</div>



<p><strong>Step 3: Managing Complex Conversations with Context</strong></p>

<p>Pydantic AI provides access to message exchange during an agent run. These messages can be used both to continue a coherent conversation and to understand how an agent performed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from pydantic_ai import Agent, RunContext
from pydantic import BaseModel

class UserPreferences(BaseModel):
    user_id: str
    preferences: dict
    conversation_history: list = []
    session_data: dict = {}

class PersonalizedResponse(BaseModel):
    response: str
    personalization_used: list[str]
    recommendations: list[str] = []

# Agent with persistent context
personal_assistant = Agent(
    'openai:gpt-4o',
    deps_type=UserPreferences,
    result_type=PersonalizedResponse,
    system_prompt="""You are a personal assistant. Use the user's preferences 
    and conversation history to provide personalized responses."""
)

@personal_assistant.tool
async def save_preference(preference_key: str, preference_value: str, ctx: RunContext[UserPreferences]) -&gt; str:
    """Save a user preference for future conversations."""
    ctx.deps.preferences[preference_key] = preference_value
    return f"Saved preference: {preference_key} = {preference_value}"

@personal_assistant.tool
async def get_user_history(ctx: RunContext[UserPreferences]) -&gt; list:
    """Retrieve the user's conversation history."""
    return ctx.deps.conversation_history[-5:] # Last 5 interactions

async def personalized_conversation():
    # Initialize user context
    user_context = UserPreferences(
        user_id="user123",
        preferences={"communication_style": "friendly", "interests": ["tech", "books"]},
        conversation_history=["Asked about Python tutorials", "Interested in AI development"]
    )

    result = await personal_assistant.run(
        "I want to learn more about web development",
        deps=user_context
    )

    print(f"Response: {result.data.response}")
    print(f"Personalization used: {result.data.personalization_used}")
    print(f"Recommendations: {result.data.recommendations}")

</code></pre>

</div>



<p><strong>What's happening under the hood?</strong></p>

<p>When <strong><em>personal_assistant.run()</em></strong> is called with _ <strong>deps=user_context</strong> :_</p>

<ul>
<li>The AI's system prompt immediately gets access to the context implied by <strong><em>user_context</em></strong>.</li>
<li>If the AI decides it needs to save a preference (e.g, if the user says "I like dark mode"), it can call the <strong><em>save_preference</em></strong> tool, and that tool will receive the same <strong><em>user_context</em></strong> object (specifically, <strong><em>ctx.deps</em></strong> ). Any changes made within the tool will persist in that <strong><em>user_context</em></strong> instance for the duration of the run.</li>
<li>Similarly, if the AI wants to recall past interactions, it can call <strong><em>get_user_history</em></strong> , which will access <strong>ctx.deps.conversation_history</strong>.</li>
</ul>

<p>This powerful pattern allows you to build stateful AI agents that truly understand and adapt to individual users over time. No more "who are you again?" moments.</p>




<p>Pydantic AI isn't just a library; it's a philosophy for building robust and intuitive AI Solutions. So, what are you waiting for? Dive in, experiment, and start building the next generation of intelligent agents that truly understand the world and the users around them.</p>

