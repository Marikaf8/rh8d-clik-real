## Accesso al codice

Il codice completo è disponibile su richiesta.

Contattare l’autore:
📧 email: marikafasanelli1222@gmail.com 

oppure

👉 contattami tramite GitHub (Issues o messaggio)

# RH8D CLIK Real Control (ROS2 Jazzy)

Questo repository contiene i nodi ROS2 sviluppati per il controllo della mano robotica RH8D su sistema reale nell’ambito della tesi:

**“Inversione Cinematica e Controllo in Forza di una Mano Robotica Sottoattuata per la Manipolazione di Tessuti”**

Il sistema implementa una pipeline per il controllo della mano reale basata su **Closed Loop Inverse Kinematics (CLIK)**, integrando la comunicazione con gli attuatori Dynamixel e i sensori di contatto fingertip.

⚠️ Il progetto è stato sviluppato e testato su **ROS2 Jazzy**.

---

## 🧠 Descrizione del sistema

Il sistema consente il controllo diretto della mano robotica RH8D su hardware reale, includendo:

- controllo cinematico delle dita tramite CLIK

- invio comandi ai motori Dynamixel

- acquisizione dati dai sensori fingertip

- controllo in forza tramite feedback dei sensori

- correzione automatica dei comandi motore durante la presa

La pipeline è progettata per garantire coerenza tra simulazione e sistema reale.

---

## ⚙️ Architettura

```text

fingertip_sensors_node

        ↓

/sensor_state

        ↓

finger_ik_controller_node

        ↓

/desired_position

        ↓

hand_driver_node

        ↓

Dynamixel motors (RH8D)

```

## 📂 Descrizione dei nodi

**finger_ik_controller_node.cpp**

Nodo principale del controllo.

Funzioni:

* Implementa il controllo CLIK su sistema reale
* Riceve il target della presa
* Calcola la configurazione articolare delle dita
* Genera i comandi per i motori
* Riceve il feedback dei sensori fingertip
* Gestisce il controllo in forza nella fase di mantenimento della presa

Input:

* /finger_target
* /sensor_state

Output:

* /desired_position

Servizi:

* /execute_finger_ik
* /start_force_hold
* /stop_force_hold
⸻

**hand_driver_node.cpp**

Nodo di interfaccia hardware.

Funzioni:

* Comunica con i motori Dynamixel
* Riceve i comandi dal controllore
* Invia i segnali ai motori reali
* Gestisce il feedback degli attuatori

Input:

* /desired_position
⸻

**fingertip_sensors_node.cpp**

Nodo per la gestione dei sensori di contatto.

Funzioni:

* Acquisisce dati dai sensori fingertip
* Pubblica informazioni di contatto
* Supporta il controllo in forza

Output:

* /sensor_state

⸻

## ✋ Gestione del controllo in forza

Il controllo in forza è gestito nel nodo:

*finger_ik_controller_node.cpp*

I dati di forza vengono acquisiti dal nodo:

*fingertip_sensors_node.cpp*

Il nodo fingertip_sensors_node.cpp legge i sensori fingertip reali e pubblica le misure di forza sul topic:

*/sensor_state*

Il nodo finger_ik_controller_node.cpp riceve queste informazioni, filtra i valori di forza e li associa ai motori della mano. Durante la fase di mantenimento della presa, il sistema confronta la forza misurata con una forza desiderata e corregge i comandi inviati ai motori.

In questo modo:

* se la forza è troppo bassa, il sistema chiude leggermente di più le dita
* se la forza è troppo alta, il sistema apre leggermente le dita
* se viene superato un limite di sicurezza, il comando viene corretto per evitare una forza eccessiva

Il controllo in forza viene attivato tramite servizio:

***ros2 service call /start_force_hold std_srvs/srv/Trigger "{}"***

e disattivato tramite:

***ros2 service call /stop_force_hold std_srvs/srv/Trigger "{}"***

## 🚀 Come eseguire il sistema

**1. Creare workspace ROS2**

*mkdir -p ~/rh8d_ws/src*

*cd ~/rh8d_ws/src*

**2. Clonare il repository**

*git clone <URL_REPOSITORY>*

**3. Installare dipendenze**

*git clone https://github.com/Vanvitelli-Robotics/uclv-dynamixel-utils*

*git clone https://github.com/Vanvitelli-Robotics/uclv-seed-robotics-ros-interfaces*

**4. Build**

*cd ~/rh8d_ws*

*colcon build*

**5. Sorgente ambiente**

*source install/setup.bash*

**6. Avvio driver della mano**

*ros2 launch uclv_seed_robotics_ros finger_ik.launch.py*

Questo comando avvia i driver della mano reale. Senza questo passaggio il sistema non può ricevere comandi.

**7. Impostare il target per il grasp**

*ros2 topic pub -r 5 /finger_target geometry_msgs/msg/Point "{x: 0.0, y: -0.01, z: 0.0}"*

Questo comando pubblica il punto target che le dita devono raggiungere durante la fase di presa.

**8. Avviare il calcolo CLIK**

*ros2 service call /execute_finger_ik std_srvs/srv/Trigger "{}"*

Questo comando avvia il calcolo del controllo cinematico CLIK e permette la chiusura delle dita verso il target impostato.

**9. Attivare il controllo in forza**

*ros2 service call /start_force_hold std_srvs/srv/Trigger "{}"*

Questo comando attiva la modalità di mantenimento della presa tramite feedback dei sensori fingertip.

**10. Disattivare il controllo in forza**

*ros2 service call /stop_force_hold std_srvs/srv/Trigger "{}"*

Questo comando disattiva il controllo in forza.

## ✋ Apertura della mano reale

Per riportare la mano in configurazione aperta:

**1. Build e sorgente ambiente**

*colcon build && source install/setup.bash*

**2. Avvio driver della mano**

*ros2 launch uclv_seed_robotics_ros finger_ik.launch.py*

**3. Pubblicazione posizione desiderata dei motori**

*ros2 topic pub /desired_position uclv_seed_robotics_ros_interfaces/msg/MotorPositions "{ids: [31, 32, 33, 34, 35, 36, 37, 38], positions: [100, 2000, 2000, 100, 100, 100, 100, 100]}"*

Questo comando invia direttamente ai motori una configurazione di apertura della mano.
