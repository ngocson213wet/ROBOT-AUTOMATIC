# =========================================================
# SUPER FACE AI ULTIMATE
# Face + Emotion + Voice + Ollama Brain
# MAC VERSION FINAL
# =========================================================

import cv2
import mediapipe as mp
import speech_recognition as sr
import ollama
import os
import time

# =========================================================
# MAC SPEAK
# =========================================================

def speak(text):

    print("\nAI:", text)

    # macOS voice
    os.system(f'say "{text}"')

# =========================================================
# OLLAMA AI BRAIN
# =========================================================

def ask_ai(user_text, emotion):

    prompt = f"""
You are a smart friendly AI assistant.

Current user emotion:
{emotion}

User said:
{user_text}

Reply naturally, friendly, short and helpful.
"""

    response = ollama.chat(

        model='phi3',

        messages=[
            {
                'role': 'user',
                'content': prompt
            }
        ]
    )

    return response['message']['content']

# =========================================================
# MICROPHONE
# =========================================================

recognizer = sr.Recognizer()

recognizer.dynamic_energy_threshold = True
recognizer.energy_threshold = 300
recognizer.pause_threshold = 0.8

mic = sr.Microphone()

# =========================================================
# CALIBRATE MIC
# =========================================================

with mic as source:

    print("Calibrating microphone...")

    recognizer.adjust_for_ambient_noise(
        source,
        duration=2
    )

print("Microphone ready!")

# =========================================================
# MEDIAPIPE FACE
# =========================================================

mp_face_mesh = mp.solutions.face_mesh

face_mesh = mp_face_mesh.FaceMesh(
    max_num_faces=1,
    refine_landmarks=True
)

# =========================================================
# CAMERA
# =========================================================

cap = cv2.VideoCapture(0)

# =========================================================
# START
# =========================================================

print("======================================")
print(" SUPER FACE AI READY ")
print("======================================")
print("ENTER = TALK")
print("Q = QUIT")
print("======================================")

emotion = "Neutral 😐"

# =========================================================
# MAIN LOOP
# =========================================================

while True:

    ret, frame = cap.read()

    if not ret:
        break

    # mirror camera
    frame = cv2.flip(frame, 1)

    h, w, c = frame.shape

    # RGB
    rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

    # face process
    results = face_mesh.process(rgb)

    color = (255,255,255)

    # =====================================================
    # FACE DETECT
    # =====================================================

    if results.multi_face_landmarks:

        for face_landmarks in results.multi_face_landmarks:

            landmarks = face_landmarks.landmark

            # =================================================
            # FACE BOX
            # =================================================

            x_list = []
            y_list = []

            for lm in landmarks:

                x_list.append(int(lm.x * w))
                y_list.append(int(lm.y * h))

            xmin = min(x_list)
            xmax = max(x_list)

            ymin = min(y_list)
            ymax = max(y_list)

            cv2.rectangle(
                frame,
                (xmin, ymin),
                (xmax, ymax),
                (0,255,0),
                2
            )

            # =================================================
            # LANDMARKS
            # =================================================

            for lm in landmarks:

                x = int(lm.x * w)
                y = int(lm.y * h)

                cv2.circle(
                    frame,
                    (x,y),
                    1,
                    (0,255,255),
                    -1
                )

            # =================================================
            # FACE ANALYSIS
            # =================================================

            mouth_width = abs(
                landmarks[61].x -
                landmarks[291].x
            )

            mouth_height = abs(
                landmarks[13].y -
                landmarks[14].y
            )

            eye_open = abs(
                landmarks[159].y -
                landmarks[145].y
            )

            # =================================================
            # HAPPY
            # =================================================

            if mouth_width > 0.12 and mouth_height > 0.02:

                emotion = "Happy 😀"

                description = """
You look happy and energetic.
"""

                color = (0,255,0)

            # =================================================
            # SURPRISED
            # =================================================

            elif mouth_height > 0.05 and eye_open > 0.02:

                emotion = "Surprised 😲"

                description = """
You look surprised today.
"""

                color = (0,255,255)

            # =================================================
            # SAD
            # =================================================

            elif eye_open < 0.012:

                emotion = "Sad 😢"

                description = """
You seem tired or sad.
"""

                color = (255,0,0)

            # =================================================
            # ANGRY
            # =================================================

            elif eye_open < 0.009:

                emotion = "Angry 😠"

                description = """
You look serious or angry.
"""

                color = (0,100,255)

            # =================================================
            # NEUTRAL
            # =================================================

            else:

                emotion = "Neutral 😐"

                description = """
You look calm and relaxed.
"""

                color = (255,255,255)

            # =================================================
            # SHOW EMOTION
            # =================================================

            cv2.putText(
                frame,
                f"Emotion: {emotion}",
                (20,50),
                cv2.FONT_HERSHEY_SIMPLEX,
                1,
                color,
                2
            )

            # =================================================
            # SHOW DESCRIPTION
            # =================================================

            y0 = 100

            for line in description.splitlines():

                cv2.putText(
                    frame,
                    line,
                    (20,y0),
                    cv2.FONT_HERSHEY_SIMPLEX,
                    0.7,
                    color,
                    2
                )

                y0 += 30

    # =====================================================
    # SHOW CAMERA
    # =====================================================

    cv2.imshow("SUPER FACE AI", frame)

    # =====================================================
    # KEYBOARD
    # =====================================================

    key = cv2.waitKey(1)

    # =====================================================
    # ENTER = TALK
    # =====================================================

    if key == 13:

        try:

            with mic as source:

                print("\nAI is listening...")

                recognizer.adjust_for_ambient_noise(
                    source,
                    duration=0.3
                )

                audio = recognizer.listen(
                    source,
                    timeout=5,
                    phrase_time_limit=5
                )

            # =================================================
            # USER SPEECH
            # =================================================

            user_text = recognizer.recognize_google(audio)

            print("\nUSER:", user_text)

            # =================================================
            # ASK OLLAMA
            # =================================================

            ai_response = ask_ai(
                user_text,
                emotion
            )

            # =================================================
            # AI SPEAK
            # =================================================

            speak(ai_response)

            # anti echo
            time.sleep(1)

        except Exception as e:

            print("\nERROR:", e)

            speak("Sorry, I could not hear you")

    # =====================================================
    # EXIT
    # =====================================================

    if key == ord('q'):

        break

# =========================================================
# CLOSE
# =========================================================

cap.release()

cv2.destroyAllWindows()
