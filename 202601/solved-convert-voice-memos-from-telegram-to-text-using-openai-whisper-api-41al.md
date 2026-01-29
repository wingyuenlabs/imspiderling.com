---
Title: Solved: Convert Voice Memos from Telegram to Text using OpenAI Whisper API
Description: 
Author: Darian Vance
Date: 2026-01-29T21:08:54.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  🚀 Executive Summary
</h3>

<p><strong>TL;DR:</strong> This project solves the problem of unstructured voice memos in Telegram by creating a Python bot that automatically transcribes them. It uses the Telegram Bot API to receive voice notes and the OpenAI Whisper API to convert them into searchable, copy-pasteable text, significantly boosting efficiency.</p>

<h4>
  
  
  🎯 Key Takeaways
</h4>

<ul>
<li>The solution integrates <code>python-telegram-bot</code> for message handling, <code>pydub</code> with <code>ffmpeg</code> for Ogg Opus to MP3 audio conversion, and the <code>openai</code> library for Whisper API transcription.</li>
<li>Secure management of API keys is achieved using <code>python-dotenv</code> to load <code>TELEGRAM\_BOT\_TOKEN</code> and <code>OPENAI\_API\_KEY</code> from a <code>config.env</code> file, preventing hardcoding.</li>
<li>Temporary audio files (OGA and MP3) are downloaded, processed, and then reliably cleaned up using <code>os.remove</code> within a <code>finally</code> block to ensure resource management.</li>
</ul>

<h1>
  
  
  Convert Voice Memos from Telegram to Text using OpenAI Whisper API
</h1>

<p>Alright, team. Darian here. Let’s talk about efficiency. I used to leave myself voice memos on the go—quick thoughts, reminders, even mini-debug sessions while walking the dog. The problem? They’d pile up in my Telegram “Saved Messages,” becoming a black hole of unstructured audio. Listening back to find one specific thought was a huge time sink. This little project changed that. Now, I just send a voice note to a bot, and a few seconds later, I get a clean text transcription back. It’s searchable, copy-pasteable, and has genuinely saved me a couple of hours a week.</p>

<p>This isn’t just a gimmick; it’s a powerful way to bridge the gap between spoken ideas and actionable, written data. Let’s build it.</p>

<h3>
  
  
  Prerequisites
</h3>

<p>Before we dive in, make sure you have the following ready. We’re all busy, so getting this sorted out first will make the process much smoother.</p>

<ul>
<li>
<strong>A Telegram Bot Token:</strong> You can get this from the BotFather on Telegram. Just start a chat with him, create a new bot, and he’ll give you an API token.</li>
<li>
<strong>An OpenAI API Key:</strong> You’ll need an account on the OpenAI platform. Grab your API key from your account dashboard.</li>
<li>
<strong>Python Environment:</strong> A working Python 3.8+ installation.</li>
<li>
<strong>FFmpeg:</strong> This is a crucial dependency for audio processing. You’ll need to install it on your system. A quick search for “install ffmpeg on [your OS]” will get you there. Pydub, the library we’ll use, depends on it.</li>
</ul>

<h3>
  
  
  The Guide: Step-by-Step
</h3>

<p>I’ll skip the standard virtual environment setup (<code>venv</code>, etc.) since you likely have your own workflow for that. Let’s jump straight to the logic. You’ll need to install a few Python libraries. Run your package installer for <code>python-telegram-bot</code>, <code>openai</code>, <code>python-dotenv</code>, and <code>pydub</code>.</p>

<h4>
  
  
  Step 1: Environment and Configuration
</h4>

<p>First rule of production: never hardcode secrets. We’ll store our API keys in a <code>config.env</code> file. Create a file with that name in your project directory and add your keys like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>TELEGRAM_BOT_TOKEN="YOUR_TELEGRAM_TOKEN_HERE"
OPENAI_API_KEY="YOUR_OPENAI_KEY_HERE"
</code></pre>

</div>



<p>Now, let’s start our Python script. We’ll call it <code>transcriber\_bot.py</code>. We’ll begin by importing the necessary libraries and loading our environment variables.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import os
import logging
from dotenv import load_dotenv
from telegram import Update
from telegram.ext import Application, MessageHandler, filters, ContextTypes
from openai import OpenAI
from pydub import AudioSegment

# Load environment variables from config.env
load_dotenv('config.env')

# Setup basic logging
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# Initialize OpenAI client
openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
</code></pre>

</div>



<h4>
  
  
  Step 2: The Telegram Bot Core Logic
</h4>

<p>Next, we’ll set up the main structure of our bot. This involves creating an <code>Application</code> instance and adding a <code>MessageHandler</code>. We specifically want to filter for voice messages, so we’ll use <code>filters.VOICE</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>async def handle_voice_message(update: Update, context: ContextTypes.DEFAULT_TYPE) -&gt; None:
    # This is where the magic will happen. We'll fill this in next.
    await update.message.reply_text("Processing your voice memo...")
    # (Future steps go here)

def main() -&gt; None:
    """Start the bot."""
    telegram_token = os.getenv("TELEGRAM_BOT_TOKEN")
    if not telegram_token:
        logger.error("TELEGRAM_BOT_TOKEN not found in environment variables!")
        return

    application = Application.builder().token(telegram_token).build()

    # Add a handler for voice messages
    application.add_handler(MessageHandler(filters.VOICE, handle_voice_message))

    # Start the Bot
    logger.info("Bot is starting...")
    application.run_polling()

if __name__ == '__main__':
    main()
</code></pre>

</div>



<p>This boilerplate code sets up a listener. When the bot receives a voice message, it will call our <code>handle\_voice\_message</code> function.</p>

<h4>
  
  
  Step 3: Downloading and Converting the Audio
</h4>

<p>Telegram voice messages usually come in the Ogg Opus audio codec (<code>.oga</code> format). Whisper API works best with more standard formats like MP3 or WAV. This is where <code>pydub</code> and <code>ffmpeg</code> shine. We’ll download the file, then convert it.</p>

<p>Let’s flesh out the <code>handle\_voice\_message</code> function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>async def handle_voice_message(update: Update, context: ContextTypes.DEFAULT_TYPE) -&gt; None:
    """Downloads, converts, and transcribes a voice message."""
    file_id = update.message.voice.file_id
    try:
        # 1. Download the file
        voice_file = await context.bot.get_file(file_id)

        # We create temporary file paths
        oga_path = f'{file_id}.oga'
        mp3_path = f'{file_id}.mp3'

        await voice_file.download_to_drive(oga_path)
        logger.info(f"Downloaded voice file to {oga_path}")

        # 2. Convert OGA to MP3
        audio = AudioSegment.from_ogg(oga_path)
        audio.export(mp3_path, format="mp3")
        logger.info(f"Converted {oga_path} to {mp3_path}")

        # (Transcription step comes next)

    except Exception as e:
        logger.error(f"An error occurred: {e}")
        await update.message.reply_text("Sorry, I couldn't process that voice memo.")
    finally:
        # 4. Clean up the temporary files
        if os.path.exists(oga_path):
            os.remove(oga_path)
        if os.path.exists(mp3_path):
            os.remove(mp3_path)
        logger.info("Cleaned up temporary files.")
</code></pre>

</div>



<blockquote>
<p><strong>Pro Tip:</strong> In my production setups, I handle file paths more robustly, often using a dedicated <code>/tmp</code> or temporary directory structure. For this example, creating files in the local directory is fine, but always be mindful of where you’re writing data, especially in a containerized environment. Cleaning up files in a <code>finally</code> block ensures they get deleted even if an error occurs.</p>
</blockquote>

<h4>
  
  
  Step 4: Transcribing with OpenAI Whisper
</h4>

<p>With our MP3 file ready, sending it to OpenAI is straightforward. We’ll use the <code>client.audio.transcriptions.create</code> method.</p>

<p>Let’s add the transcription logic into our <code>handle\_voice\_message</code> function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>async def handle_voice_message(update: Update, context: ContextTypes.DEFAULT_TYPE) -&gt; None:
    """Downloads, converts, and transcribes a voice message."""
    file_id = update.message.voice.file_id
    oga_path = f'{file_id}.oga'
    mp3_path = f'{file_id}.mp3'

    try:
        await update.message.reply_text("Processing your voice memo...")
        voice_file = await context.bot.get_file(file_id)
        await voice_file.download_to_drive(oga_path)

        audio = AudioSegment.from_ogg(oga_path)
        audio.export(mp3_path, format="mp3")

        # 3. Send to Whisper API for transcription
        with open(mp3_path, "rb") as audio_file:
            transcription = openai_client.audio.transcriptions.create(
                model="whisper-1",
                file=audio_file
            )

        transcribed_text = transcription.text
        logger.info(f"Transcription successful: {transcribed_text}")

        # 4. Reply to the user
        await update.message.reply_text(f"Transcription:\n\n{transcribed_text}", parse_mode='HTML')

    except Exception as e:
        logger.error(f"An error occurred: {e}")
        await update.message.reply_text("Sorry, I couldn't process that voice memo.")
    finally:
        # 5. Clean up
        if os.path.exists(oga_path):
            os.remove(oga_path)
        if os.path.exists(mp3_path):
            os.remove(mp3_path)
        logger.info("Cleaned up temporary files for " + file_id)
</code></pre>

</div>



<p>And that’s the complete loop! The bot receives a voice note, downloads it, converts it, sends it to Whisper, and replies with the text.</p>

<h3>
  
  
  Common Pitfalls
</h3>

<p>Here are a few places I’ve tripped up in the past. Hopefully, you can avoid them.</p>

<ul>
<li>
<strong><code>ffmpeg</code> Not Found:</strong> The most common issue. The <code>pydub</code> library is just a Python wrapper around the <code>ffmpeg</code> command-line tool. If <code>ffmpeg</code> isn’t installed and available in your system’s PATH, <code>pydub</code> will fail. The error message is usually pretty clear about this.</li>
<li>
<strong>API Key Errors:</strong> Double-check your <code>config.env</code> file. A typo in the variable name or a misplaced quote can lead to authentication failures. Make sure the file is in the same directory you’re running the script from, or provide an absolute path to it.</li>
<li>
<strong>File Size Limits:</strong> The OpenAI Whisper API has a file size limit (currently 25 MB). For a simple voice memo bot, this is rarely an issue. But if you were adapting this for longer audio, you’d need to implement chunking—splitting the audio into smaller pieces and processing them sequentially.</li>
</ul>

<h3>
  
  
  Conclusion
</h3>

<p>You now have a fully functional, private transcription service. This pattern is incredibly versatile. You could modify it to save transcriptions to a database, send them to a Notion page, or create a Jira ticket. It’s a fantastic building block for automating any workflow that starts with a spoken idea.</p>

<p>Happy building,<br><br>
Darian Vance<br><br>
Senior DevOps Engineer, TechResolve</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" alt="Darian Vance" width="758" height="289"></a></p>

<p>👉 <a href="https://wp.me/pbK4oa-fR" rel="noopener noreferrer">Read the original article on TechResolve.blog</a></p>




<p>☕ <strong>Support my work</strong>  </p>

<p>If this article helped you, you can buy me a coffee:  </p>

<p>👉 <a href="https://buymeacoffee.com/darianvance" rel="noopener noreferrer">https://buymeacoffee.com/darianvance</a></p>

