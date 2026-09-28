# Data-Driven Fault Detection in DC Motor Circuits

> **Edge-based predictive maintenance system for DC motors using LTspice digital-twin simulation, Motor Current Signature Analysis (MCSA), machine learning, and embedded hardware validation.**

## Overview

This project develops a **data-driven fault detection and predictive maintenance architecture for DC motor systems**.

The central idea is to identify motor faults from their **electrical current signatures** rather than relying on invasive mechanical sensors such as accelerometers.

The system combines:

* **LTspice-based digital-twin modelling**
* Automated SPICE simulation using **shell scripting**
* Synthetic generation of large-scale motor fault datasets
* **Motor Current Signature Analysis (MCSA)**
* Time-domain current feature extraction
* Comparative evaluation of machine-learning classifiers
* **Dependency-free C++ deployment**
* An **8-bit microcontroller edge node**
* **ACS712 current sensing**
* Physical DC motor hardware testing

The complete workflow follows:

```text
DC Motor + Drive Model
        ↓
LTspice Digital Twin
        ↓
Automated Fault Injection
        ↓
Large Synthetic Dataset
        ↓
Current-Signature Feature Extraction
        ↓
ML Model Training & Comparison
        ↓
Model Selection
        ↓
C++ Embedded Deployment
        ↓
ACS712 Current Sensing
        ↓
Real-Time Edge Fault Classification
```

The paper reports a dataset containing approximately **47,000 cases** spanning multiple thermal, electrical, and mechanical fault conditions.

---

# Problem Statement

DC motors are widely used in industrial automation, robotics, aerospace systems, and consumer equipment. Motor degradation can originate from electrical, thermal, or mechanical problems and may eventually result in system failure.

Traditional condition-monitoring approaches can require:

* Physical vibration sensors
* Accelerometers
* Extensive data acquisition hardware
* Cloud-based processing
* Physically induced motor faults for training-data collection

These approaches can become expensive or difficult to deploy on constrained systems.

This project explores a different approach:

> **Can physical motor faults be identified using only electrical current measurements and lightweight machine-learning models running locally on a microcontroller?**

The project addresses this through **Motor Current Signature Analysis combined with physics-based simulation and edge ML**.

---

# Objectives

The main objectives were:

1. Develop an LTspice model of a DC motor and its drive circuitry.
2. Create a digital-twin-based simulation environment for fault generation.
3. Automate large-scale simulation using shell scripting.
4. Generate synthetic data representing different motor fault conditions.
5. Extract meaningful electrical features from motor current.
6. Compare multiple machine-learning architectures.
7. Select models suitable for constrained edge deployment.
8. Convert the selected model into dependency-free C++.
9. Interface an 8-bit microcontroller with an ACS712 current sensor.
10. Validate the concept using a physical DC motor testbed.
11. Perform fault classification locally without requiring cloud connectivity.

---

# System Architecture

```text
                 ┌───────────────────────┐
                 │   DC Motor Digital    │
                 │        Twin           │
                 │       LTspice         │
                 └───────────┬───────────┘
                             │
                             ▼
                 ┌───────────────────────┐
                 │ Automated Fault       │
                 │ Injection + Simulation │
                 │    Shell Pipeline     │
                 └───────────┬───────────┘
                             │
                             ▼
                 ┌───────────────────────┐
                 │ Synthetic Motor       │
                 │ Current Dataset       │
                 └───────────┬───────────┘
                             │
                             ▼
                 ┌───────────────────────┐
                 │ Feature Extraction    │
                 │ Iavg / Irms / Imax    │
                 │ Imin / Iripple        │
                 └───────────┬───────────┘
                             │
                             ▼
                 ┌───────────────────────┐
                 │ ML Model Comparison   │
                 └───────────┬───────────┘
                             │
                             ▼
                 ┌───────────────────────┐
                 │ Embedded C++ Model    │
                 └───────────┬───────────┘
                             │
                             ▼
        ┌─────────────────────────────────────┐
        │       Physical Hardware Node        │
        │                                     │
        │ ACS712 → 8-bit MCU → Fault Class   │
        │            Detection                │
        └─────────────────────────────────────┘
```

The architecture is designed to bridge the gap between **simulation-based model development and real-world embedded deployment**.

---

# 1. LTspice Digital Twin

The first stage of the project is the creation of a **digital representation of the DC motor and its drive circuitry**.

The simulated system includes:

* DC motor model
* MOSFET-based drive circuit
* Electrical parameters
* Mechanical-load-related parameters
* Current measurement node

The drive circuit uses an **IRLHM620 MOSFET** in the simulation model.

The digital twin provides a controllable environment in which different fault conditions can be introduced without physically damaging a motor.

---

# 2. Automated Dataset Generation

Manually creating thousands of SPICE simulations would be inefficient.

To solve this, a **shell scripting pipeline** was developed around LTspice.

The automation process:

```text
Modify Simulation Parameters
          ↓
Generate LTspice Netlist
          ↓
Run Simulation from Command Line
          ↓
Generate Transient Response
          ↓
Extract Electrical Parameters
          ↓
Store Dataset
          ↓
Repeat
```

The pipeline introduces randomized mathematical scaling factors into the simulation parameters and executes large numbers of transient simulations automatically.

The simulations cover a **0–50 ms transient window**, with the study generating approximately **50,000 simulation conditions** before the resulting dataset used for evaluation is described as approximately **47,000 cases**.

---

# 3. Fault Modelling

Nine different fault classes were modelled across thermal, electrical, and mechanical operating conditions.

Fault conditions were represented by modifying the appropriate electrical or mechanical parameters of the digital twin.

Examples include:

### Thermal Fault

Temperature-related parameter variation was introduced to reproduce changes in motor electrical behaviour.

### Duty-Cycle Fault

PWM duty-cycle conditions were modified to represent abnormal drive operation.

### Resistance Fault

Motor resistance was varied to represent changes in electrical characteristics.

### Reduced-Voltage Fault

The source voltage was modified to represent abnormal supply conditions.

### Bearing Fault

The baseline viscous drag coefficient was scaled by approximately **5× to 19×** to represent increased mechanical resistance.

### Vibration Fault

A high-frequency oscillating mechanical-load component was introduced, with the reported vibration frequency reaching approximately **60–200 Hz** during fault modelling.

These parameter changes produce corresponding changes in the electrical current waveform.

---

# 4. Motor Current Signature Analysis

The key principle of the project is **Motor Current Signature Analysis (MCSA)**.

Instead of directly measuring mechanical parameters, the system observes the motor's electrical current.

The reasoning is:

```text
Mechanical / Electrical Fault
          ↓
Change in Motor Behaviour
          ↓
Change in Electrical Load
          ↓
Change in Motor Current
          ↓
Distinct Current Signature
          ↓
Fault Classification
```

This makes the motor current itself a useful diagnostic signal.

The project therefore avoids depending on dedicated vibration sensors for the primary fault-detection mechanism.

---

# 5. Feature Extraction

For every 50 ms simulation window, current-related features were extracted from the simulated Hall-effect current-sensing node.

The primary features were:

| Feature   | Description              |
| --------- | ------------------------ |
| `Iavg`    | Average current          |
| `Irms`    | Root-mean-square current |
| `Imax`    | Maximum current          |
| `Imin`    | Minimum current          |
| `Iripple` | Current ripple           |

`Iripple` was specifically calculated to capture higher-frequency current variations associated with mechanical chattering and other changes in the current waveform.

The extracted features form the input space for the machine-learning classifier.

---

# 6. Dataset Behaviour

The generated dataset showed measurable differences between fault classes.

The reported values included:

* Average current approximately **1.58–3.38 A**
* RMS current approximately **1.94–3.47 A**
* Temperature approximately **27–99°C**
* Source voltage approximately **9.1–12 V**

Fault Class 6 produced the highest reported current values, while Fault Class 3 showed comparatively lower current values.

The vibration-related class showed a reported vibration amplitude of approximately **0.005** and frequency around **63 Hz** in the analysed dataset.

These variations demonstrate that the simulated fault conditions produce distinguishable electrical signatures.

---

# 7. Machine Learning Pipeline

The extracted features were supplied to multiple machine-learning models.

The study evaluated:

* Logistic Regression
* Decision Tree
* Random Forest
* Gradient Boosting
* AdaBoost
* XGBoost

The purpose was not simply to maximize classification accuracy, but to identify models that could also be translated into a constrained embedded environment.

```text
Current Features
      ↓
Feature Dataset
      ↓
Train / Evaluate Models
      ↓
Accuracy
F1 Score
Precision
Recall
      ↓
Select Suitable Model
      ↓
Embedded Implementation
```

The results showed strong performance from tree-based approaches.

---

# 8. Model Evaluation

The reported classification results include:

| Model               | Reported Accuracy |
| ------------------- | ----------------: |
| XGBoost             |            0.9991 |
| Random Forest       |            0.9985 |
| Decision Tree       |            0.9984 |
| Logistic Regression |            0.9646 |
| AdaBoost            |            0.3354 |

The paper reports that **XGBoost achieved the highest classification accuracy**, while Random Forest and Decision Tree also produced very high classification performance.

The analysis attributes the strong performance of tree-based models to their ability to establish nonlinear decision boundaries between different fault signatures.

---

# 9. Why Tree-Based Models?

The extracted current features do not necessarily form simple linear clusters.

For example, combinations such as:

```text
Iavg
 +
Irms
 +
Iripple
```

can produce nonlinear relationships between different fault conditions.

Tree-based classifiers can divide this feature space using decision boundaries that better represent these nonlinear relationships.

The paper also observed that Logistic Regression struggled with overlapping fault profiles, while AdaBoost showed severe degradation when interacting with intentionally introduced variations such as temperature fluctuations and duty-cycle jitter.

---

# 10. Embedded Deployment

A major objective was to avoid making the final diagnostic system dependent on a desktop computer, Python runtime, or cloud platform.

The selected model was translated into **dependency-free C++** for embedded execution.

The deployment concept is:

```text
ACS712 Current Sensor
        ↓
Analog Current Signal
        ↓
8-bit Microcontroller
        ↓
Feature Calculation
        ↓
Embedded ML Model
        ↓
Fault Classification
        ↓
Local Output
```

This creates an **offline edge-diagnostic system** capable of performing classification locally.

---

# 11. Physical Hardware Testbed

To investigate the transition from simulation to physical hardware, a small diagnostic testbed was developed around a **12 V brushed DC motor**.

The hardware node contains:

* 12 V brushed DC motor
* 8-bit microcontroller
* ACS712 5 A current sensor
* IRLZ44N logic-level MOSFET
* Low-side switching configuration
* Flyback diode
* Parallel ceramic capacitors

The MOSFET controls the motor, while the ACS712 provides the electrical current measurement required for the diagnostic algorithm.

---

# 12. Hardware Signal Path

The physical system can be represented as:

```text
12 V Supply
    │
    ▼
DC Motor
    │
    ▼
ACS712 Current Sensor
    │
    ▼
8-bit Microcontroller
    │
    ├── Current Feature Extraction
    │
    └── Embedded ML Classification
              │
              ▼
       Fault Classification
```

The MOSFET provides low-side switching of the motor.

Because a motor is an inductive load, switching generates transient effects that must be controlled.

---

# 13. Hardware Protection & Noise Suppression

One important difference between simulation and physical hardware is the presence of:

* Electromagnetic interference
* Inductive kickback
* Analog measurement noise
* Physical motor imperfections

The hardware therefore includes a **flyback diode** and parallel ceramic capacitors to suppress unwanted transients and improve signal stability.

This hardware-level conditioning was important for maintaining current-signal integrity during embedded testing.

---

# 14. Sim-to-Real Architecture

One of the central engineering challenges is the difference between ideal simulated signals and real motor measurements.

The project therefore follows a:

> **Simulation → Model → Hardware → Calibration → Classification**

workflow.

```text
                 SIMULATION
                     │
                     ▼
          LTspice Digital Twin
                     │
                     ▼
             Synthetic Dataset
                     │
                     ▼
              ML Classification
                     │
                     ▼
             Embedded C++ Model
                     │
                     ▼
                 HARDWARE
                     │
                     ▼
             ACS712 Measurement
                     │
                     ▼
           Physical Calibration
                     │
                     ▼
          Local Fault Classification
```

The paper describes mathematical transfer/calibration techniques to account for differences between synthetic current features and real hardware measurements.

---

# 15. Real-Time Embedded Output

The physical testbed was also used to display real-time values through an LCD interface.

The embedded system therefore provides a local diagnostic interface rather than depending on cloud connectivity.

The final concept is:

```text
Motor
 ↓
Current Sensor
 ↓
Microcontroller
 ↓
Feature Extraction
 ↓
ML Classification
 ↓
LCD / Local Diagnostic Output
```

The paper reports that the standalone system performed offline fault classification using the localized signal from the ACS712 sensor.

---

# 16. Engineering Workflow

The project followed a hardware + simulation + data-science workflow:

### Step 1 — Define Motor System

Develop the DC motor and MOSFET-drive model.

### Step 2 — Build Digital Twin

Implement the electrical and parameterized motor behaviour in LTspice.

### Step 3 — Define Fault Conditions

Introduce controlled changes representing thermal, electrical, and mechanical faults.

### Step 4 — Automate Simulation

Use shell scripting to modify parameters and repeatedly execute LTspice simulations.

### Step 5 — Generate Dataset

Collect current and operating-condition data from the simulations.

### Step 6 — Extract Features

Calculate:

```text
Iavg
Irms
Imax
Imin
Iripple
```

### Step 7 — Train ML Models

Evaluate multiple classification architectures.

### Step 8 — Select Model

Compare classification performance and embedded suitability.

### Step 9 — Convert to C++

Translate the selected model into dependency-free embedded code.

### Step 10 — Build Hardware Node

Interface the ACS712 sensor and MOSFET-controlled DC motor with an 8-bit MCU.

### Step 11 — Calibrate

Account for differences between simulated and physical current measurements.

### Step 12 — Perform Local Classification

Run the diagnostic model directly on the embedded node.

---

# 17. Key Technical Concepts

This project combines several engineering domains:

### Power Electronics

* MOSFET switching
* Low-side motor drive
* Inductive-load protection
* Flyback suppression

### Embedded Systems

* 8-bit microcontroller
* ADC/current measurement
* Real-time feature extraction
* Embedded C++

### Circuit Simulation

* LTspice
* SPICE netlists
* Transient analysis
* Parameterized fault modelling

### Data Engineering

* Automated simulation
* Synthetic dataset generation
* Feature extraction
* Large-scale batch processing

### Machine Learning

* Classification
* Decision Trees
* Random Forest
* Gradient Boosting
* XGBoost
* Logistic Regression
* AdaBoost

### Predictive Maintenance

* Motor condition monitoring
* Motor Current Signature Analysis
* Fault classification
* Sim-to-real validation

---

# 18. Technical Stack

| Category            | Technologies                                                                            |
| ------------------- | --------------------------------------------------------------------------------------- |
| Circuit Simulation  | LTspice                                                                                 |
| Automation          | Shell scripting                                                                         |
| Motor Modelling     | DC motor + MOSFET drive                                                                 |
| Current Sensing     | ACS712 5 A                                                                              |
| Embedded Controller | 8-bit MCU                                                                               |
| Embedded Language   | C++                                                                                     |
| Machine Learning    | Decision Tree, Random Forest, Gradient Boosting, AdaBoost, XGBoost, Logistic Regression |
| Hardware Switching  | IRLZ44N MOSFET                                                                          |
| Protection          | Flyback diode, ceramic capacitors                                                       |
| Output              | LCD                                                                                     |

---

# 19. Results

The project demonstrated that time-domain motor-current features can provide useful information for differentiating motor fault conditions.

Key reported outcomes include:

* Approximately **47,000 evaluated cases**
* Nine modelled fault classes
* Current-signature-based feature extraction
* Successful comparison of multiple ML architectures
* XGBoost accuracy of **0.9991**
* Random Forest accuracy of **0.9985**
* Decision Tree accuracy of **0.9984**
* Dependency-free C++ embedded deployment
* Physical validation using a 12 V brushed DC motor
* Offline classification using an ACS712 current sensor

These results support the feasibility of combining **digital-twin simulation, current sensing, lightweight ML, and embedded edge computing** for localized motor diagnostics.

---

# 20. Key Engineering Challenge

The most important challenge was the **Sim-to-Real gap**.

Simulation provides:

* Controlled conditions
* Repeatability
* Large datasets
* Easy fault injection

Real hardware introduces:

* Sensor noise
* EMI
* Motor-to-motor variation
* Component tolerances
* Mechanical imperfections
* Switching transients

The project addresses this through hardware noise suppression and calibration/transfer techniques between the synthetic and physical feature domains.

---

# 21. What Makes the Project Interesting

The project combines three normally separate stages:

```text
Physics-Based Simulation
          +
Machine Learning
          +
Embedded Hardware
```

Instead of collecting every possible fault physically, the digital twin is used to generate controlled fault data.

Instead of requiring a powerful computer for inference, the trained model is converted into embedded C++.

Instead of using vibration sensors as the primary diagnostic input, the system uses the motor's **electrical current signature**.

This produces a compact architecture aimed at **localized, offline predictive maintenance**.

---

# 22. Applications

The architecture can be relevant to applications involving electrically driven machinery, including:

* Industrial motor condition monitoring
* Factory automation
* Robotics
* Remote machinery
* Aerospace systems
* Embedded predictive maintenance
* Resource-constrained diagnostic systems

The paper specifically discusses industrial and aerospace applications where local processing and reduced dependence on cloud infrastructure can be valuable.

---

# 23. Future Scope

The paper identifies further physical validation and sim-to-real calibration as important future development areas.

Potential next steps include:

* Larger physical motor testbeds
* More real-world fault datasets
* Improved sensor calibration
* Domain adaptation between simulated and physical data
* More extensive embedded benchmarking
* Continuous condition monitoring
* Integration with industrial maintenance systems

The paper specifically proposes recording physical baseline measurements and applying mathematical scaling factors so that synthetic-model thresholds better correspond to real hardware measurements.

---

# 24. Repository Structure

A possible repository structure:

```text
Data-Driven-Fault-Detection/
│
├── README.md
│
├── LTspice/
│   ├── motor_model.asc
│   ├── drive_circuit.asc
│   └── netlists/
│
├── Simulation/
│   ├── fault_models/
│   ├── transient_runs/
│   └── results/
│
├── Automation/
│   └── simulation_scripts/
│
├── Dataset/
│   ├── raw/
│   └── processed/
│
├── Feature_Extraction/
│   └── current_features/
│
├── ML/
│   ├── DecisionTree/
│   ├── RandomForest/
│   ├── XGBoost/
│   └── evaluation/
│
├── Embedded/
│   ├── C++/
│   └── microcontroller/
│
├── Hardware/
│   ├── circuit/
│   └── testbench/
│
└── Documentation/
    ├── figures/
    └── results/
```

---

# 25. Skills Demonstrated

This project demonstrates practical experience in:

* LTspice circuit simulation
* SPICE netlist generation
* Shell scripting and simulation automation
* Digital-twin modelling
* Motor-drive circuits
* MOSFET switching
* Current sensing
* Analog signal acquisition
* Embedded C++
* Microcontroller-based diagnostics
* Feature engineering
* Machine-learning classification
* Predictive maintenance
* Motor Current Signature Analysis
* Hardware testing
* EMI/noise mitigation
* Sim-to-real model validation

---

# 26. Project Takeaway

This project demonstrates an end-to-end engineering workflow:

> **Model → Simulate → Automate → Generate Data → Extract Features → Train → Optimize → Deploy → Sense → Classify**

The key idea is to transform **electrical current measurements into actionable motor-condition information** while keeping the final diagnostic intelligence close to the physical system.

By combining LTspice-based synthetic data generation, MCSA, lightweight machine learning, and embedded C++ deployment, the project explores a practical route toward **offline edge predictive maintenance for constrained motor-control environments**.
