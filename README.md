# Facial-Recognize-in-Python
## Descrição/Description:
[PT-BR]
Este projeto implementa um sistema de detecção de rostos em tempo real utilizando a biblioteca OpenCV em Python. O sistema é capaz de identificar rostos através da webcam e exibir uma barra de carregamento enquanto cada rosto é processado.
O projeto pode ser executado em qualquer sistema que suporte Python e tenha uma webcam conectada. Ao ser iniciado, ele abrirá uma janela de visualização, onde os rostos detectados serão destacados e as barras de carregamento serão exibidas. O usuário pode encerrar o programa pressionando a tecla 'q'.

Funcionalidades:
- Detecção de Rostos: Utiliza um classificador em cascata pré-treinado para identificar rostos em tempo real.
- Barra de Carregamento: Cada rosto detectado exibe uma barra de progresso que indica o tempo necessário para o "carregamento" do rosto. Esse tempo é configurável e simula um processo de preparação (ex. "desinfecção").
- Congelamento de Rostos: Após o término do carregamento, os rostos são "congelados" temporariamente. Um mecanismo é implementado para reiniciar o processo após todos os rostos na tela terem completado o carregamento.
- Interface Visual: As detecções são visualmente representadas na tela com retângulos ao redor dos rostos, além da barra de carregamento que muda de tamanho conforme o progresso.

[ENG]
This project implements a real-time face detection system using the OpenCV library in Python. The system is capable of identifying faces through the webcam and displaying a loading bar while each face is processed. The project can be executed on any system that supports Python and has a connected webcam. When started, it will open a preview window where detected faces will be highlighted and loading bars will be displayed. The user can terminate the program by pressing the 'q' key.

Features:
- Face Detection: Utilizes a pre-trained cascade classifier to identify faces in real-time.
- Loading Bar: Each detected face displays a progress bar indicating the time required for the "loading" of the face. This time is configurable and simulates a preparation process (e.g., "disinfection").
- Freezing Faces: After the loading is complete, the faces are temporarily "frozen." A mechanism is implemented to restart the process after all faces on screen have completed the loading.
- Visual Interface: The detections are visually represented on the screen with rectangles around the faces, along with a loading bar that changes size according to the progress.

## Versão/Version:
[PT-BR]
O código foi desenvolvido na versão Python 3.12, porém ele pode ser usado em qualquer versão a partir da versão Python 3.6.

[ENG]
The code was developed in Python 3.12, but it can be used in any version starting from Python 3.6.

## Bibliotécas/Libraries:
[PT-BR] 
As bíbliotecas usadas foram:
- OpenCV
- time

[ENG]
The libraries used were:
- Open CV
- time

## Materiais/Materials:
[PT-BR]
- Câmera webcam Full HD Logitech C920
- Qualquer sistema operacional que suporte o Python

[ENG]
- Full HD Logitech C920 Webcam
- Any operating system that supports Python

## Passo a Passo/How To Do It:
[PT-BR]
### 1. PREPARAÇÃO DO AMBIENTE
- Instale o Python:
  - Baixe e instale a versão mais recente do Python (recomendado 3.6 ou superior) do site oficial: python.org. O instalador geralmente já inclui o IDLE.

### 2. ABRINDO O IDLE
- Abra o IDLE:
  - Após a instalação, abra o IDLE do Python. Você pode encontrar o IDLE no menu de programas do Python ou digitando "IDLE" na busca do seu sistema operacional.

### 3. INSTALAÇÃO DAS BIBLIOTECAS:
- Abra o Terminal do IDLE:

  - No IDLE, vá até o menu e clique em File > New File para abrir um novo editor.
Instale as Bibliotecas Necessárias:

- No novo arquivo, cole o seguinte código para instalar as bibliotecas necessárias:
-----------------------------------------------------------------------------------------------------------------------------------------
import pip

import sys

#Função para instalar pacotes

def install(package):

    pip.main(['install', package])
    
#Instalação do OpenCV e NumPy

install('opencv-python')

install('numpy')

-----------------------------------------------------------------------------------------------------------------------------------------

- Salve e Execute o Código:

  - Salve o arquivo como install_packages.py.
  - Execute o código pressionando F5 ou indo em Run > Run Module. Isso instalará o OpenCV e o NumPy.

### 4. IMPLEMENTAÇÃO DO CÓDIGO PRINCIPAL:
- Crie um Novo Arquivo para o Código:
  - No IDLE, novamente clique em File > New File.

- Cole o Código de Detecção de Rostos:
  - Copie e cole o seguinte código no novo arquivo:
-----------------------------------------------------------------------------------------------------------------------------------------
import cv2

import time

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')

cap = cv2.VideoCapture(0)

loading_start_times = {}

completed_faces = set()

frozen_faces = {}

loading_duration = 2  # Duration in seconds

reset_delay = 2

reset_time = None

reset_in_progress = False


while True:

    ret, frame = cap.read()
    
    if not ret:
    
        break

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
    faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30))

    current_time = time.time()

    if reset_in_progress:
    
        if current_time - reset_time >= reset_delay:
        
            loading_start_times.clear()
            
            completed_faces.clear()
            
            frozen_faces.clear()
            
            reset_time = None
            
            reset_in_progress = False
            
    else:
    
        all_faces_frozen = True
        
        for (x, y, w, h) in faces:
        
            face_id = (x, y, w, h)

            if face_id not in loading_start_times:
            
                loading_start_times[face_id] = current_time

            elapsed_time = current_time - loading_start_times[face_id]
            
            if elapsed_time < loading_duration:
            
                all_faces_frozen = False
                
                cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 0, 0), 2)

                bar_width = w
                
                bar_height = 20
                
                bar_x = x
                
                bar_y = y + h + 5
                
                progress = min(1, elapsed_time / loading_duration)

                cv2.rectangle(frame, (bar_x, bar_y), (bar_x + bar_width, bar_y + bar_height), (0, 0, 0), -1)
                
                cv2.rectangle(frame, (bar_x, bar_y), (bar_x + int(bar_width * progress), bar_y + bar_height), (0, 255, 0), -1)
                
            else:
            
                frozen_faces[face_id] = current_time + reset_delay

        faces_ids_on_screen = set((x, y, w, h) for (x, y, w, h) in faces)
        
        for face_id in list(completed_faces):
        
            if face_id not in faces_ids_on_screen:
            
                completed_faces.remove(face_id)
                
                if face_id in loading_start_times:
                
                    del loading_start_times[face_id]
                    
                if face_id in frozen_faces:
                
                    del frozen_faces[face_id]

        if all_faces_frozen:
        
            cv2.putText(frame, "Todas as barras carregadas", (50, 50), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 255), 2)
            
            if reset_time is None:
            
                reset_time = current_time
                
                reset_in_progress = True
                
        else:
        
            cv2.imshow('Face Detection with Loading Bar', frame)

    for face_id, end_time in list(frozen_faces.items()):
    
        if current_time < end_time:
        
            (x, y, w, h) = face_id
            
            cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)

    cv2.imshow('Face Detection with Loading Bar', frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
    
        break

cap.release()

cv2.destroyAllWindows()

-----------------------------------------------------------------------------------------------------------------------------------------

- Salve o Arquivo:

  - Salve o arquivo como face_detection.py.

### 5. EXECUÇÃO DO CÓDIGO:
- Execute o Código:
  - No IDLE, pressione F5 ou vá em Run > Run Module para executar o código.

### 6. INTERAÇÃO COM O PROGRAMA:
- Use a Webcam:

  - Certifique-se de que a webcam esteja funcionando. Quando o programa estiver em execução, ele abrirá uma janela mostrando a detecção de rostos e as barras de carregamento.

- Finalizar o Programa:
  - Pressione a tecla 'q' para sair do programa.

### CONCLUSÃO:
Agora você tem um projeto funcional de detecção de rostos com barra de carregamento usando o IDLE! Você pode personalizar e expandir esse projeto conforme necessário. Se precisar de mais alguma coisa, é só avisar!

[ENG]
### 1. ENVIRONMENT SETUP
- Install Python:
  - Download and install the latest version of Python (recommended 3.6 or higher) from the official website: python.org. The installer usually includes IDLE.

### 2. OPENING IDLE
- Open IDLE:
  - After installation, open Python's IDLE. You can find IDLE in the Python program menu or by typing "IDLE" in your operating system's search bar.

### 3. INSTALLING LIBRARIES:
- Open the IDLE Terminal:

  - In IDLE, go to the menu and click on File > New File to open a new editor.

Install the Necessary Libraries:

- In the new file, paste the following code to install the required libraries:
-----------------------------------------------------------------------------------------------------------------------------------------
import pip

import sys

#Function to install packages

def install(package):

    pip.main(['install', package])

#Installation of OpenCV and NumPy

install('opencv-python')

install('numpy')

-----------------------------------------------------------------------------------------------------------------------------------------

- Save and Run the Code:

  - Save the file as install_packages.py.
  - Execute the code by pressing F5 or going to Run > Run Module. This will install OpenCV and NumPy.

### 4. IMPLEMENTING THE MAIN CODE
- Create a New File for the Code:
  - In IDLE, click again on File > New File.

- Paste the Face Detection Code:
  - Copy and paste the following code into the new file:
-----------------------------------------------------------------------------------------------------------------------------------------
import cv2

import time

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')

cap = cv2.VideoCapture(0)

loading_start_times = {}

completed_faces = set()

frozen_faces = {}

loading_duration = 2  # Duration in seconds

reset_delay = 2

reset_time = None

reset_in_progress = False

while True:

    ret, frame = cap.read()
    if not ret:
    
        break

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
    faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30))

    current_time = time.time()

    if reset_in_progress:
    
        if current_time - reset_time >= reset_delay:
        
            loading_start_times.clear()
            
            completed_faces.clear()
            
            frozen_faces.clear()
            
            reset_time = None
            
            reset_in_progress = False
            
    else:
    
        all_faces_frozen = True
        
        for (x, y, w, h) in faces:
        
            face_id = (x, y, w, h)
            

            if face_id not in loading_start_times:
            
                loading_start_times[face_id] = current_time

            elapsed_time = current_time - loading_start_times[face_id]
            
            if elapsed_time < loading_duration:
            
                all_faces_frozen = False
                
                cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 0, 0), 2)

                bar_width = w
                
                bar_height = 20
                
                bar_x = x
                
                bar_y = y + h + 5
                
                progress = min(1, elapsed_time / loading_duration)

                cv2.rectangle(frame, (bar_x, bar_y), (bar_x + bar_width, bar_y + bar_height), (0, 0, 0), -1)
                
                cv2.rectangle(frame, (bar_x, bar_y), (bar_x + int(bar_width * progress), bar_y + bar_height), (0, 255, 0), -1)
                
            else:
            
                frozen_faces[face_id] = current_time + reset_delay

        faces_ids_on_screen = set((x, y, w, h) for (x, y, w, h) in faces)
        
        for face_id in list(completed_faces):
        
            if face_id not in faces_ids_on_screen:
            
                completed_faces.remove(face_id)
                
                if face_id in loading_start_times:
                
                    del loading_start_times[face_id]
                    
                if face_id in frozen_faces:
                
                    del frozen_faces[face_id]

        if all_faces_frozen:
        
            cv2.putText(frame, "Todas as barras carregadas", (50, 50), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 255), 2)
            
            if reset_time is None:
            
                reset_time = current_time
                
                reset_in_progress = True
                
        else:
        
            cv2.imshow('Face Detection with Loading Bar', frame)

    for face_id, end_time in list(frozen_faces.items()):
    
        if current_time < end_time:
        
            (x, y, w, h) = face_id
            
            cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)

    cv2.imshow('Face Detection with Loading Bar', frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
    
        break

cap.release()

cv2.destroyAllWindows()

-----------------------------------------------------------------------------------------------------------------------------------------

- Save the File:

  - Save the file as face_detection.py.

### 5. RUNNING THE CODE:
- Run the Code:
  - In IDLE, press F5 or go to Run > Run Module to execute the code.

### 6. INTERACTION WITH THE PROGRAM:
- Use the Webcam:

  - Make sure the webcam is functioning. When the program is running, it will open a window displaying face detection and loading bars.

- Exit the Program:
  - Press the 'q' key to exit the program.

### CONCLUSION:
Now you have a functional face detection project with a loading bar using IDLE! You can customize and expand this project as needed. If you need anything else, just let me know!
