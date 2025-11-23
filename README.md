# Covid-19-Project-using-Numpy-and-Pandas

ABSTRACT 
With the growing concerns over unauthorized access in secure facilities and organizations, there is a compelling need for an intelligent surveillance system that can detect intruders in real time. This project proposes a hybrid intrusion detection framework combining the speed and efficiency of YOLOv8 (You Only Look Once) for object and face detection with the deep learning capabilities of Convolutional Neural Networks (CNNs) for face classification. The system determines whether a detected individual is an authorized person by comparing the face with a pre-stored facial dataset. If the person is unrecognized (unauthorized), the system immediately triggers an alert mechanism via email or SMS.

The system is trained and evaluated using CrowdHuman for dense human detection and OpenVGun6 dataset to handle scenarios involving potential threats like firearms. This hybrid approach allows robust classification even in challenging conditions such as varying lighting, occlusion, and crowded scenes. The result is a highly scalable, accurate, and real-time security surveillance solution.

Introduction:
Security is paramount in today's corporate, educational, and defense infrastructures. Conventional security systems often rely on manual monitoring or rule-based motion detection, which are prone to errors and inefficiency. With the increasing sophistication of intruders, there's a need for smarter surveillance — particularly those which can detect and identify intrusions based on faces and behaviors.

This project introduces an AI-powered surveillance system combining YOLOv8 for real-time detection and a CNN for classification. The YOLO module detects faces in video frames, while the CNN verifies if the face matches with a known authorized dataset. If no match is found, an alert is sent via email or SMS through API integration (like Twilio or SMTP).

By leveraging facial image datasets and YOLO’s object detection, the model captures the frame, extracts the face, and classifies it using a CNN trained on authorized personnel. Alerts are generated based on classification confidence. The model supports real-time processing from CCTV inputs.

Overview :

Goal: Detect intruders by combining object detection (person/weapon) with behavior analysis; raise an alert if weapon detected or behavior is suspicious. Datasets: CrowdHuman (persons) + OpenImages (Gun/Knife/Rifle). Output: annotated frames + alert status. 

We are making this project so that we can able to identify the suspicious behaviour  which is recorded by different kinds of cameras through which we are using this CrowdHuman and Openv6 Images where we are combining these two datasets and converting into YOLO format and likewise we are applying the fusion of LBPH and CNN to classify those behaviours i.e. normal and suspicious. 

Implementation:

Collect & Prepare Data
Clone CrowdHuman; unzip images; parse .odgt → YOLO labels.
FiftyOne: open-images-v6 with classes ["Gun","Knife","Rifle"], max_samples=5000; export images + labels.
Create YAMLs, 80/20 split; optional 25% sampling.
Train Detectors (YOLOv8n)
CrowdHuman: !yolo task=detect mode=train model=yolov8n.pt epochs=36 imgsz=640 batch=16
OpenImages: !yolo task=detect mode=train model=yolov8n.pt epochs=50 imgsz=640 batch=16
Run Inference & Crop
Load best.pt for both models; detect; crop person ROIs.
Train Hybrid Classifier
ResNet-18 backbone (frozen optional) + LBPH → fusion → sigmoid.
Loss: BCEWithLogits; Optim: AdamW; Scheduler: CosineAnnealingLR; WeightedRandomSampler for class balance.
Decision & Reporting
Intruder if weapon present or behavior suspicious; save overlays and CSV, emit alert flag.

WORKFLOW :
<img width="517" height="866" alt="image" src="https://github.com/user-attachments/assets/7bfbcd85-30d3-41b8-b948-282ff2911a42" />

MODEL ARCHITECTURE :
<img width="1762" height="771" alt="image" src="https://github.com/user-attachments/assets/b843bceb-fd29-4fa1-8b25-7ee99c05eefd" />

Data Preprocessing :

Detection (two YOLOv8n models) 

Person Detector: CrowdHuman (Person): trained 36 epochs; qualitative tests show stable person boxes in validation samples.

Weapon Detector: OpenImages (Weapons): trained 50 epochs; predicted Gun/Knife/Rifle reliably in held-out images.

Inference returns boxes, class, confidence for each frame. 

 Behavior Classifier (CNN + LBPH Fusion) 

Backbone: torchvision.models.resnet18 (ImageNet weights, fc=Identity → 512-D). 

LBPH branch: 10-D → FC(64) + ReLU + Dropout. 

Fusion: concat(512,64)=576 → FC(128) → FC(1 logit). 

Transforms: Resize(224), ColorJitter (train), Normalize(ImageNet). 








