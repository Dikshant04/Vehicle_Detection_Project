# Vehicle Speed Detection System

Real-time vehicle speed estimation using YOLOv8 object detection, ByteTrack tracking, and perspective transformation for accurate speed calculation from video footage.

## Overview

This project implements an automated vehicle speed detection system that processes video footage to detect, track, and calculate vehicle speeds. The system uses perspective transformation to convert pixel movements into real-world distance measurements, enabling accurate speed calculations in km/h.

## Key Features

- **YOLOv8 Object Detection**: High-accuracy vehicle detection using pre-trained YOLOv8x model
- **ByteTrack Multi-Object Tracking**: Consistent vehicle tracking across video frames
- **Perspective Transformation**: Converts 2D video coordinates to real-world measurements
- **Speed Calculation**: Real-time speed estimation in km/h based on distance and time
- **Zone-based Filtering**: Processes only vehicles within defined region of interest
- **Visual Annotations**: Bounding boxes, tracking trails, and speed labels

## Implementation Details

### Detection Pipeline
```python
# YOLOv8 model initialization
model = YOLO("yolov8x.pt")

# Detection with confidence filtering
detections = detections[detections.confidence > CONFIDENCE_THRESHOLD]
detections = detections[polygon_zone.trigger(detections)]
```

### Tracking System
```python
# ByteTrack tracker initialization
byte_track = sv.ByteTrack(
    frame_rate=video_info.fps, 
    track_activation_threshold=CONFIDENCE_THRESHOLD
)

# Update tracking with new detections
detections = byte_track.update_with_detections(detections=detections)
```

### Perspective Transformation
```python
class ViewTransformer:
    def __init__(self, source: np.ndarray, target: np.ndarray):
        self.m = cv2.getPerspectiveTransform(source, target)
    
    def transform_points(self, points: np.ndarray) -> np.ndarray:
        transformed_points = cv2.perspectiveTransform(reshaped_points, self.m)
        return transformed_points.reshape(-1, 2)
```

### Speed Calculation
```python
# Calculate speed based on coordinate history
coordinate_start = coordinates[tracker_id][-1]
coordinate_end = coordinates[tracker_id][0]
distance = abs(coordinate_start - coordinate_end)
time = len(coordinates[tracker_id]) / video_info.fps
speed = distance / time * 3.6  # Convert to km/h
```

## Configuration Parameters

- **Model**: YOLOv8x (yolov8x.pt)
- **Resolution**: 1280px for detection
- **Confidence Threshold**: 0.3
- **IOU Threshold**: 0.5 for NMS
- **Tracking**: ByteTrack with frame rate adaptation
- **Speed Buffer**: FPS/2 frames minimum for speed calculation

## Region of Interest Setup

```python
# Define perspective transformation coordinates
SOURCE = np.array([
    [1252, 787],   # Top-left
    [2298, 803],   # Top-right  
    [5039, 2159],  # Bottom-right
    [-550, 2159]   # Bottom-left
])

TARGET = np.array([
    [0, 0],
    [TARGET_WIDTH - 1, 0],
    [TARGET_WIDTH - 1, TARGET_HEIGHT - 1],
    [0, TARGET_HEIGHT - 1]
])
```

## Visual Output

The system provides comprehensive visual feedback:
- **Bounding Boxes**: Vehicle detection boundaries
- **Tracking Trails**: Vehicle movement paths
- **Speed Labels**: Real-time speed display in km/h
- **Zone Overlay**: Region of interest visualization

## Usage

```python
# Process video for speed detection
with sv.VideoSink(TARGET_VIDEO_PATH, video_info) as sink:
    for frame in frame_generator:
        # Detection and tracking pipeline
        result = model(frame, imgsz=MODEL_RESOLUTION)
        detections = sv.Detections.from_ultralytics(result)
        
        # Apply filters and tracking
        detections = detections[polygon_zone.trigger(detections)]
        detections = byte_track.update_with_detections(detections)
        
        # Calculate speeds and annotate
        annotated_frame = apply_annotations(frame, detections, speeds)
        sink.write_frame(annotated_frame)
```

## Requirements

```
supervision
ultralytics
opencv-python
numpy
tqdm
```

## Applications

- **Traffic Monitoring**: Automated speed enforcement systems
- **Traffic Analysis**: Road usage and speed pattern studies  
- **Safety Assessment**: Identifying speeding violations
- **Urban Planning**: Traffic flow optimization data
