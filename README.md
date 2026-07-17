# Speech-to-text
!pip install --upgrade gradio typer click
import gradio as gr
from google import genai
from google.genai import types
# from gtts import gTTS # Temporarily removed due to dependency conflict
import tempfile

client=genai.Client(
    api_key=""
)

def extract_text(file):
  if file is None:
    return "please upload a file first.",None
  uploaded_file_ref=client.files.upload(file=file.name)
  response=client.models.generate_content(
      model="gemini-2.5-flash",
      contents=[uploaded_file_ref,"Extract all text exactly as written."]
  )
  return response.text,None

def summarize(file):
  if file is None:
    return "please upload a file first.",None
  uploaded_file_ref=client.files.upload(file=file.name)
  response=client.models.generate_content(
      model="gemini-2.5-flash",
      contents=[uploaded_file_ref,"Summarize this document in simple language."]
  )
  return response.text,None

def answers_question(file,question):
  if file is None:
    return "please upload a file first.",None
  if not question.strip():
    return "please enter a question.",None
  uploaded_file_ref=client.files.upload(file=file.name)
  response=client.models.generate_content(
      model="gemini-2.5-flash",
      contents=[uploaded_file_ref,f"Answer the following question using ONLY this document .\n\nQuestion:\n{question}"]
  )
  return response.text,None

def audio_summary(file):
  # gTTS is currently incompatible with the required Gradio dependencies
  # To enable audio summary, you'll need to use an alternative text-to-speech library
  # that does not conflict with Gradio's dependencies.
  # Examples: pyttsx3, or direct integration with cloud TTS APIs.
  return "Audio summary disabled due to dependency conflict. Please use an alternative TTS library.", None

with gr.Blocks() as demo:
  gr.Markdown("Gemini Document Analyzer")
  with gr.Row():
    with gr.Column(scale=1):
      file=gr.File(label="Upload PDF/Image")
    with gr.Column(scale=2):
      gr.Markdown("### choose an Action")
      with gr.Row():
        btn_extract=gr.Button("Extract Text",variant="secondary")
        btn_summarize=gr.Button("Summarize Document",variant="secondary")
        # btn_audio=gr.Button("Generate Audio Summary",variant="secondary") # Temporarily removed
      gr.Markdown("---")
      gr.Markdown("### ? Document Q&A")
      question=gr.Textbox(
          label="Ask a specific question about the document",
          placeholder="Type your question here..."
      )
      btn_qa=gr.Button("Answer Question",variant="primary")
      gr.Markdown("---")
      gr.Markdown("### Outputs")
      with gr.Row():
        output=gr.Textbox(label="Text output",lines=12,scale=2)
        audio=gr.Audio(label="Audio output",scale=1, visible=False) # Make audio output invisible

  btn_extract.click(
      fn=extract_text,
      inputs=[file],
      outputs=[output,audio]
  )
  btn_summarize.click(
      fn=summarize,
      inputs=[file],
      outputs=[output,audio]
  )
  # Temporarily remove audio button click event
  # btn_audio.click(
  #     fn=audio_summary,
  #     inputs=[file],
  #     outputs=[output,audio]
  # )
  btn_qa.click(
      fn=answers_question,
      inputs=[file, question],
      outputs=[output,audio]
  )

demo.launch(debug=True)
