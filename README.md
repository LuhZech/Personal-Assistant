# Personal-Assistant

import speech_recognition as sr
import pyttsx3
from _datetime import datetime
import wikipedia
import webbrowser
import os
import time
import subprocess
import wolframalpha
import json
import requests


engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voice','voices[1].id')

def speak(text):
    engine.say(text)
    engine.runAndWait()

def wishMe():
    hour = datetime.now().hour
    if hour>=0 and hour<12:
        speak("Good Morning")
        print("Good Morning")
    elif hour>=12 and hour<18:
        speak("Good Afternoon")
        print("Good Afternoon")
    else:
        speak("Good Night")
        print("Good Night")

def takeCommand():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        audio = r.listen(source)

        try:
            statement = r.recognize_google(audio,language='en-in')
            print(f"user said:{statement}\n")

        except Exception as e:
            speak("Pardon me, please say that again")
            return "None"
        return statement

print("Loading your AI personal assistant G-One")
speak("Loading your AI personal assistant G_One")
wishMe()

SCOPES = ['https://www.googleapis.com/auth/calendar.readonly']

def main():
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())

        service = build('calendar', 'v3', credentials=creds)

        #Call the calendar API
        now = datetime.datetime.utcnow().isoformat() + 'Z'
        #Z indicates UTC time
        print('Getting the upcoming  10 events')
        events_result = service.events().list(calendarId='primary',
                                              timeMin=now, maxResults=10,
                                              singleEvents=True,
                                              orderBy='startTime').execute()
        events = events_result.get('items', [])

        if not events:
            print('No upcoming events found.')
        for event in events:
            start = event['start'].get('dateTime', event['start'].get(date))
            print(start, event['summary'])


if __name__=='__main__':


    while True:
        speak("Tell me how can I help you now?")
        statement = takeCommand().lower()
        if statement == 0:
            continue

        if "good bye" in statement or "ok bye" in statement or "stop" in staticmethod:
            speak('your personal assistant G-one is shutting down, Bood bye')
            print('your personal assistant  G-one is shutting down, Good Bye')
            break

        if 'wikipedia' in statement:
            speak('Searching Wikipedia...')
            statement = statement.replace("wikipedia", "")
            results = wikipedia.summary(statement, sentences= 3)
            speak("According do Wikipedia")
            print(results)
            speak(results)

        elif 'open youtube' in statement:
            webbrowser.open_new_tab("https://www.youtube.com.br")
            speak("Youtube is open now")
            time.sleep(5)

        elif 'open google' in statement:
            webbrowser.open_new_tab("https://www.google.com.br")
            speak("Google is open now")
            time.sleep(5)

        elif 'open yahoo' in statement:
            webbrowser.open_new_tab("https://www.yahoo.com.br")
            speak("Yahoo is open now")
            time.sleep(5)

        elif 'time' in statement:
            strTime = datetime.now().strftime("%H:%M:%S")
            speak(f"the time is {strTime}")

        elif 'search' in statement:
            statement = statement.replace("search", "")
            webbrowser.open_new_tab(statement)
            time.sleep(5)

        elif 'ask' in statement:
            speak('I can answer to computational and geographical questions and what question do you want to ask now')
            question = takeCommand()
            app_id = "Paste your unique ID here"
            client = wolframalpha.Client('R2K75H-7ELALHR35X')
            res = client.query(question)
            answer = next(res.results).text
            speak(answer)
            print(answer)

        elif "weather" in statement:
            api_key = "Apply your unique ID"
            base_url = "https://api.openweathermap,org/data/2.5/weather?"
            speak("what is the city name?")
            city_name = takeCommand()
            complete_url = base_url + "appid =" + api_key + "&q =" +city_name
            response = requests.get(complete_url)
            x = response.json()

            if x["cod"]!= "404":
                y = x["main"]
                current_temperature = y["temp"]
                current_humidity = y["humidity"]
                z = x["weather"]
                weather_description = z[0]["description"]
                speak("Temperature in kelvin unit is " +
                      str(current_temperature) +
                      "\n humidity in percentage is "+
                      str(current_humidity)+
                      "\n description " +
                      str(weather_description))
                print("Temperature in kelvin unit = " +
                      str(current_temperature) +
                      "\n humidity (in percentage) =" +
                      str(current_humidity) +
                      "\n description =" +
                      str(weather_description))

            elif "log off" in statement or "sign out" in statement:
                speak("Ok, your Pc will log off in 10 seconds make sure you exit from"
                      "all applications")
                subprocess.call(["shutdown", "/1"])

time.sleep(3)
