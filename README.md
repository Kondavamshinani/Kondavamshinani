import 'dart:async';

import 'package:flutter/material.dart';
import 'package:google_mlkit_object_detection/google_mlkit_object_detection.dart';

class ObjectTrackingScreen extends StatefulWidget {
  @override
  _ObjectTrackingScreenState createState() => _ObjectTrackingScreenState();
}

class _ObjectTrackingScreenState extends State<ObjectTrackingScreen> {
  GoogleMlKitObjectDetector _objectDetector;
  List<DetectedObject> _detectedObjects = [];

  Completer<CameraImage> _cameraImageCompleter = Completer();

  @override
  void initState() {
    super.initState();

    _objectDetector = GoogleMlKitObjectDetector();

    // Start processing the camera feed.
    _startProcessingCameraFeed();
  }

  void _startProcessingCameraFeed() {
    CameraController _cameraController = CameraController(
      CameraLensDirection.back,
      ResolutionPreset.high,
    );

    _cameraController.initialize().then((_) {
      _cameraController.startImageStream((CameraImage cameraImage) {
        _cameraImageCompleter.complete(cameraImage);
      });
    });

    Future.wait([_cameraController.initialize(), _objectDetector.initialize()])
        .then((_) {
      _processCameraFeed();
    });
  }

  Future<void> _processCameraFeed() async {
    CameraImage cameraImage = await _cameraImageCompleter.future;

    // Detect objects in the camera feed.
    List<DetectedObject> detectedObjects = await _objectDetector.processImage(cameraImage);

    // Update the state with the detected objects.
    setState(() {
      _detectedObjects = detectedObjects;
    });

    // Schedule the next frame to be processed.
    _processCameraFeed();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Object Tracking'),
      ),
      body: Stack(
        children: [
          CameraPreview(_cameraController),
          CustomPaint(
            painter: _ObjectTrackingPainter(_detectedObjects),
          ),
        ],
      ),
    );
  }
}

class _ObjectTrackingPainter extends CustomPainter {
  final List<DetectedObject> _detectedObjects;

  _ObjectTrackingPainter(this._detectedObjects);

  @override
  void paint(Canvas canvas, Size size) {
    for (var detectedObject in _detectedObjects) {
      // Draw a bounding box around the detected object.
      canvas.drawRect(
        detectedObject.boundingBox,
        Paint()
          ..color = Colors.red
          ..strokeWidth = 2.0,
      );
    }
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => true;
}
