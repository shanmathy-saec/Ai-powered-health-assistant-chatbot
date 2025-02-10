import re
import random
import speech_recognition as sr
import pyttsx3
import tkinter as tk
from tkinter import scrolledtext


class HealthChatbot:
    def __init__(self):
        self.responses = {
            r"hi|hello|hey": ["Hello! How can I assist you with your health today?", "Hi there! How can I help you with your health?"],
            r"how are you?": ["I'm just a bot, but I'm here to help you with health-related questions!"],
            r"(.*) your name(.*)": ["I'm your health assistant bot! You can call me HealthBot.", "I am a chatbot created to assist with your health queries."],
            r"(.*) (help|assist)(.*)": ["Sure! I can help you with health information, symptoms, and more! How can I assist you?"],

            # General advice responses
            r"what should I do if I have a headache": [
                "Headaches can be caused by stress, dehydration, or other factors. Try drinking some water, resting, or taking over-the-counter pain relief like ibuprofen or acetaminophen. However, if it persists, please consult a doctor.",
                "To relieve a headache, make sure you’re hydrated, rest in a quiet place, and consider taking a mild painkiller. But if the pain is severe or frequent, it’s wise to see a doctor. Healthy foods like leafy greens and fruits may help reduce headaches."
            ],
            r"(.*) tired|fatigue(.*)": [
                "Fatigue can be caused by stress, lack of sleep, or poor nutrition. Make sure to get enough rest, stay hydrated, and eat foods rich in iron, like spinach or red meat. If the fatigue continues, it might be worth talking to a healthcare professional.",
                "If you're feeling tired, take regular breaks, stay hydrated, and maintain a balanced diet with whole grains, lean proteins, and fruits. Persistent fatigue should be checked by a healthcare provider."
            ],
            r"(.*) cold|cough|sore throat": [
                "For a common cold, rest and fluids are important. You might also consider over-the-counter cold medicine to ease symptoms. A sore throat can sometimes be relieved by gargling warm salt water. Foods like honey, ginger, and garlic can help soothe a sore throat.",
                "Cold and cough symptoms typically go away on their own, but if you're experiencing severe symptoms, please consult a doctor. Consider warm teas with honey, and steam inhalation to ease discomfort."
            ],
            r"(.*) fever(.*)": [
                "If you have a fever, make sure to rest and drink plenty of fluids. If the fever lasts for more than a couple of days, or if it’s very high, it’s a good idea to consult a healthcare professional. You can also take fever-reducing tablets like paracetamol.",
                "Stay hydrated, take fever-reducing medication like paracetamol, and rest. If the fever exceeds 103°F or persists for several days, please consult a doctor."
            ],
            r"(.*) (diabetes|high blood sugar)": [
                "Diabetes is a condition where your body either doesn't produce enough insulin or doesn't use it effectively. It's important to manage your diet and blood sugar levels. A balanced diet with low sugar and regular exercise can help. Healthy foods for diabetes include whole grains, vegetables, and lean proteins.",
                "Managing diabetes involves checking blood sugar levels regularly, eating a balanced diet, and staying active. Please consult with a healthcare provider to get a personalized plan. Common tablets include metformin."
            ],
            r"(.*) (hypertension|high blood pressure)": [
                "Hypertension can be managed through lifestyle changes such as reducing salt intake, exercising regularly, and avoiding alcohol and smoking. It’s also important to monitor your blood pressure and take prescribed medication if necessary. Healthy foods like leafy greens, bananas, and oats can help manage blood pressure.",
                "For high blood pressure, try reducing stress, eating a heart-healthy diet, and getting regular physical activity. Medication like ACE inhibitors or beta-blockers may be necessary in some cases."
            ],
            r"(.*) (heart disease|heart attack)": [
                "Heart disease symptoms include chest pain, shortness of breath, and dizziness. To reduce risk, it’s important to maintain a healthy weight, eat a balanced diet (like vegetables, fruits, whole grains), exercise regularly, and avoid smoking.",
                "If you experience chest pain, nausea, or discomfort in your arm or jaw, it’s important to seek immediate medical help as it could indicate a heart attack."
            ],
            r"(.*) (stroke)": [
                "Stroke symptoms can include sudden numbness, confusion, trouble speaking, and difficulty walking. If you experience these symptoms, seek emergency help immediately.",
                "Stroke can be prevented by controlling high blood pressure, avoiding smoking, and eating a healthy diet. Seek immediate medical attention if you suspect a stroke."
            ],
            r"(.*) (asthma)": [
                "Asthma is a chronic condition where your airways become inflamed. It can be triggered by allergens, exercise, or respiratory infections. Keep track of your symptoms and have an inhaler or medication as prescribed by your doctor.",
                "For asthma management, avoid triggers, take prescribed medications, and use your inhaler when needed. Consult your doctor for a personalized asthma action plan."
            ],
            r"(.*) (arthritis)": [
                "Arthritis involves inflammation of the joints and can cause pain and stiffness. Regular exercise, maintaining a healthy weight, and physical therapy may help alleviate symptoms. Anti-inflammatory medications may help as well.",
                "For arthritis pain relief, try anti-inflammatory medications and gentle exercise. Speak to your doctor about specific treatment plans."
            ],
            r"(.*) (muscle pain|muscle soreness)": [
                "Muscle pain can be caused by overuse, tension, or injury. Rest, hydration, and applying ice or heat may help. Over-the-counter pain relief medications like ibuprofen or acetaminophen can also alleviate discomfort.",
                "For muscle pain relief, try gentle stretching, hot or cold compresses, and anti-inflammatory medications. Foods like bananas, nuts, and leafy greens can aid muscle recovery."
            ],
            r"(.*) (leg pain)": [
                "Leg pain can be caused by a variety of factors, including muscle strain, poor circulation, or injury. Elevating your legs, resting, and using ice may help. You can also take pain relievers like ibuprofen.",
                "For leg pain, make sure to rest, elevate the leg, and apply ice if necessary. Healthy foods like salmon (rich in omega-3) can help reduce inflammation."
            ],
            r"(.*) (ear pain)": [
                "Ear pain can be caused by an infection, a blocked ear canal, or other conditions. Over-the-counter pain relievers, such as ibuprofen, can help. If the pain persists or is accompanied by fever, see a doctor.",
                "For ear pain relief, use warm compresses and pain relievers. Avoid inserting anything into the ear. Seek medical advice if symptoms persist."
            ],
            r"(.*) (running nose|nasal congestion)": [
                "A running nose can be a sign of a cold, allergies, or sinus infection. Using saline nasal spray and staying hydrated can help. Over-the-counter decongestants like pseudoephedrine may also help clear the nose.",
                "For a running nose, try using a saline nasal spray and steam inhalation. Drinking warm fluids like herbal teas can help soothe your throat."
            ],
            r"(.*) (irregular periods)": [
                "Irregular periods can be caused by stress, hormonal imbalances, or other health conditions. Maintaining a healthy weight, managing stress, and eating foods like leafy greens, whole grains, and berries may help.",
                "If you're experiencing irregular periods, it's important to consult a healthcare provider for diagnosis. A balanced diet, along with medications like birth control pills, can help regulate menstrual cycles."
            ],
            r"what is (.*) disease|what are the symptoms of (.*)": [
                "Please describe the symptoms you're experiencing, and I'll try to assist you.",
                "Could you tell me more about the condition you're referring to?",
                "I recommend consulting a healthcare provider for specific diagnosis and treatment options. I can give you general information though."
            ],
            r"(.*)": [
                "I'm sorry, I don't understand that question. Can you try asking something related to health?",
                "Could you please clarify or ask about a health-related topic?"
            ]
        }

    def respond(self, user_input):
        for pattern, response_list in self.responses.items():
            match = re.search(pattern, user_input, re.IGNORECASE)
            if match:
                response = random.choice(response_list)
                if pattern in [r"what is (.*) disease|what are the symptoms of (.*)"]:
                    return response.format(match.group(1))  # Format the response with the disease name
                return response
        return "I'm sorry, I don't quite understand. Could you please rephrase?"

    def text_to_speech(self, text):
        engine = pyttsx3.init()
        engine.setProperty('rate', 150)  # Speed of speech
        engine.setProperty('volume', 1)  # Volume level (0.0 to 1.0)
        engine.say(text)
        engine.runAndWait()

    def listen_for_input(self):
        recognizer = sr.Recognizer()
        with sr.Microphone() as source:
            print("Listening for your health query...")
            recognizer.adjust_for_ambient_noise(source)
            audio = recognizer.listen(source)
            try:
                user_input = recognizer.recognize_google(audio)
                print(f"You said: {user_input}")
                return user_input.lower()
            except sr.UnknownValueError:
                print("Sorry, I could not understand the audio. Please try again.")
                return ""
            except sr.RequestError:
                print("Sorry, there was an error with the speech recognition service.")
                return ""


class ChatbotGUI:
    def __init__(self, root, chatbot):
        self.chatbot = chatbot
        self.root = root
        self.root.title("Health Assistant Chatbot")
        self.root.geometry("600x700")
        self.root.config(bg="#f0f0f0")
        
        # Title Label
        self.title_label = tk.Label(self.root, text="Health Assistant", font=("Arial", 20, "bold"), bg="#f0f0f0")
        self.title_label.pack(pady=10)

        # Chat box
        self.chat_box = scrolledtext.ScrolledText(self.root, wrap=tk.WORD, width=70, height=20, state='disabled', font=("Arial", 12))
        self.chat_box.pack(padx=20, pady=10)

        # User input box
        self.user_input = tk.Entry(self.root, width=60, font=("Arial", 14))
        self.user_input.pack(padx=20, pady=10)

        # Send button
        self.send_button = tk.Button(self.root, text="Send", width=20, height=2, command=self.send_message, font=("Arial", 14), bg="#4CAF50", fg="white")
        self.send_button.pack(pady=5)

        # Voice input button
        self.voice_button = tk.Button(self.root, text="Speak", width=20, height=2, command=self.voice_input, font=("Arial", 14), bg="#008CBA", fg="white")
        self.voice_button.pack(pady=5)

    def send_message(self):
        user_input = self.user_input.get().lower()
        self.display_message("You: " + user_input)
        response = self.chatbot.respond(user_input)
        self.display_message("HealthBot: " + response)
        self.chatbot.text_to_speech(response)
        self.user_input.delete(0, tk.END)

    def voice_input(self):
        user_input = self.chatbot.listen_for_input()
        if user_input:
            self.display_message("You: " + user_input)
            response = self.chatbot.respond(user_input)
            self.display_message("HealthBot: " + response)
            self.chatbot.text_to_speech(response)

    def display_message(self, message):
        self.chat_box.config(state='normal')
        self.chat_box.insert(tk.END, message + "\n")
        self.chat_box.config(state='disabled')
        self.chat_box.yview(tk.END)


def main():
    chatbot = HealthChatbot()
    root = tk.Tk()
    gui = ChatbotGUI(root, chatbot)
    root.mainloop()


if __name__ == "__main__":
    main()
