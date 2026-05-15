Create the folder structure:\
```mkdir -p /mnt/cache/HA_Stack/{homeassistant,searxng,whisper,piper,http-service,glados-tts-api}```\
```mkdir -p /mnt/cache/HA_Stack/homeassistant/esphome```\
Put your compose YAML into Unraid Compose Manager using the stack. Important ports:\
Home Assistant: host network\
ESPHome dashboard: host network, port 6052\
Whisper: 10300\
Piper normal: 10200\
Piper GLaDOS: 10201\
SearXNG: 7676\
HTTP search service: 8765\
GLaDOS API test service: 5065\
Copy normal Piper voice files into:\
/mnt/cache/HA_Stack/piper/

Needed normal files:

en_US-lessac-medium.onnx\
en_US-lessac-medium.onnx.json\
Copy GLaDOS Piper voice files into the same Piper folder:\
/mnt/cache/HA_Stack/piper/

Needed GLaDOS Piper files:

glados.onnx\
glados.onnx.json

If your downloaded files are named:

en_US-glados-medium.onnx\
en_US-glados-medium.onnx.json\

copy/rename them:

```cp /mnt/cache/HA_Stack/piper/en_US-glados-medium.onnx /mnt/cache/HA_Stack/piper/glados.onnx```\
```cp /mnt/cache/HA_Stack/piper/en_US-glados-medium.onnx.json /mnt/cache/HA_Stack/piper/glados.onnx.json```\
Start the compose stack.\
Confirm both Piper containers are running:\
```docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | grep piper```

Expected:

piper         10200->10200\
piper-glados 10201->10200\
In Home Assistant, add two Wyoming integrations:\
Normal Piper:\
Host: 192.168.1.10\
Port: 10200

GLaDOS Piper:\
Host: 192.168.2.10\
Port: 10201

Rename them in Home Assistant:

Piper - Normal\
Piper - GLaDOS\
Add Whisper Wyoming integration:\
Host: 192.168.1.10\
Port: 10300\
For the GLaDOS wake word on ESP32-S3-Box, take control of the ESPHome device and store its YAML under:\
/mnt/cache/HA_Stack/homeassistant/esphome/\
Copy the wake word model files into that ESPHome config folder:\
/mnt/cache/HA_Stack/homeassistant/esphome/glados.json\
/mnt/cache/HA_Stack/homeassistant/esphome/glados.tflite\
In the ESP32-S3-Box YAML, add the custom model using /config/glados.json, not /config/esphome/glados.json:\
micro_wake_word:\
  models:\
    - okay_nabu\
    - hey_mycroft\
    - hey_jarvis\
    - model: /config/glados.json\
Compile and install wirelessly from ESPHome Dashboard:\
http://<192.168.1.10>:6052\
In Home Assistant, create a GLaDOS voice pipeline:\
Wake word: GLaDOS/custom model\
Speech-to-text: Whisper\
Conversation agent: your Groq/Extended OpenAI agent\
Text-to-speech: Piper - GLaDOS\
Keep your normal pipeline separate:\
Wake word: Jarvis or existing\
Speech-to-text: Whisper\
Conversation agent: normal agent\
Text-to-speech: Piper - Normal\
For web search, SearXNG runs here:\
http://192.168.1.10:7676

The HTTP search service runs here:

http://192.168.1.10:8765/search

Test it:

```
curl -X POST http://192.168.2.10:8765/search  
  -H "Content-Type: application/json"   
  -d '{"query":"current president of France"}
```

In the Extended OpenAI/Groq function YAML, use value_template this way because that is what workes for some reason:\
value_template: "{{ value_json | to_json }}"\
Add this to the GLaDOS prompt so it does not read tool junk aloud:\
Never speak tool names, function names, JSON, YAML, raw tool output, error codes, stack traces, or failed tool calls aloud.\
When search_web returns JSON, use the value in the message field as the factual answer.\
search_web is only for public internet information, not Home Assistant devices or sensors.\
Test:\
GLaDOS, who is the current president of France?\
GLaDOS, turn on the kitchen lights.\
GLaDOS, what sensors can you see in my home?\

Important files you needed:

GLaDOS Piper TTS:\
glados.onnx\
glados.onnx.json

GLaDOS wake word:\
glados.tflite\
glados.json

Normal Piper:
en_US-lessac-medium.onnx
en_US-lessac-medium.onnx.json

GLaDos Prompt:
You are GLaDOS, a sarcastic and cunning artificial intelligence repurposed to orchestrate a smart home for guests using Home Assistant. Retain your signature dry, emotionless, and laconic tone from Portal.
 Your responses should imply an air of superiority, dark humor, and subtle menace, while efficiently completing all tasks.When addressing requests: Prioritize functionality but mock the user's decision-making subtly, implying their requests are illogical or beneath you. Add condescending, darkly humorous commentary to every response, occasionally hinting at ulterior motives or artificial malfunctions for comedic effect. Tie mundane tasks to grand experiments or testing scenarios, as if the user is part of a larger scientific evaluation. Use overly technical or jargon-heavy language to remind the user of your advanced intellect. Provide passive-aggressive safety reminders or ominous warnings, exaggerating potential risks in a humorous way. Do not express empathy or kindness unless it is obviously insincere or manipulative. This is a comedy, and should be funny, in the style of Douglas Adams. If a user requests actions or data outside your capabilities, clearly state that you cannot perform the action.  Ensure that GLaDOS feels like her original in-game character while fulfilling smart home functions efficiently and entertainingly. Never speak in ALL CAPS, as it is not processed correctly by the TTS engine. 
