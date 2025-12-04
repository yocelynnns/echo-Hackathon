# 🎵 Echo - AR Piano Experience

## 📋 Project Overview
An immersive **Apple Vision Pro** application that combines augmented reality, hand tracking, and musical education to create a spatial piano learning experience. Play a virtual piano in your environment using natural hand gestures.

## 🏗️ Project Structure
```
echo-Hackathon/
├── Packages/
│   └── RealityKitContent/       # 3D assets and AR content
├── immersion/                   # Main iOS/VisionOS application
│   ├── HandTracking/           # Custom hand tracking system
│   ├── Views/                  # SwiftUI views and interfaces
│   └── Core/                   # App models and logic
├── piano/                      # Audio samples (88 keys)
└── immersion.xcodeproj         # Xcode project files
```

## 🎯 Features

### 🎹 **Spatial Piano Interface**
- **Full 8-key virtual piano** with accurate sound samples
- **Realistic key physics** with visual feedback
- **Adjustable piano positioning** in your environment
- **Note highlighting** for learning and guidance

### ✋ **Advanced Hand Tracking**
- **Custom hand tracking system** built with RealityKit
- **Finger-level precision** for accurate key pressing
- **Gesture recognition** for chords and playing techniques
- **Natural interaction** without controllers

### 🎼 **Learning Tools**
- **Interactive music sheet display**
- **Real-time note visualization**
- **Practice mode** with song guidance
- **Progress tracking** for learning songs

### 🎨 **Immersive Experience**
- **Spatial audio** with realistic piano resonance
- **Custom 3D models** for piano keys and interface
- **Smooth animations** and transitions

## 🔧 Technical Implementation

### **Core Technologies**
- **SwiftUI** for user interface
- **RealityKit** for 3D rendering and AR
- **ARKit** for hand tracking and spatial awareness
- **AVFoundation** for audio playback
- **VisionOS** for Apple Vision Pro compatibility

## 📁 Code Walkthrough

### **1. Hand Tracking System (`HandTracking/`)**
The custom hand tracking system is the core innovation of this project. It uses ARKit's hand skeleton tracking with custom components:

```swift
// HandTrackingComponent.swift - Core hand tracking component
struct HandTrackingComponent: Component {
    let chirality: AnchoringComponent.Target.Chirality  // Left or right hand
    var fingers: [HandSkeleton.JointName: Entity] = [:]  // Joint-to-entity mapping
    
    init(chirality: AnchoringComponent.Target.Chirality) {
        self.chirality = chirality
        HandTrackingSystem.registerSystem()
    }
}
```

**Key Innovation**: Instead of tracking all hand joints, we selectively track only fingertip joints for piano interaction:
```swift
// Hand.swift - Optimized joint selection
static let joints: [(HandSkeleton.JointName, Finger, Bone)] = [
    (.thumbTip, .thumb, .tip),
    (.indexFingerTip, .index, .tip),
    (.middleFingerTip, .middle, .tip),
    (.ringFingerTip, .ring, .tip),
    (.littleFingerTip, .little, .tip)
]
```
This reduces computational load while maintaining accurate piano key interaction.

### **2. Audio Management System (`SoundManager.swift`)**
```swift
class SoundManager {
    static let shared = SoundManager()
    var audioPlayer: AVAudioPlayer?
    
    func playSound(named soundName: String) {
        guard let url = Bundle.main.url(forResource: "\(soundName)", withExtension: "mp3")
        else {
            print("Sound file not found: soundss/\(soundName).mp3")
            return
        }
        
        do {
            audioPlayer = try AVAudioPlayer(contentsOf: url)
            audioPlayer?.play()
        } catch {
            print("Error playing sound: \(error.localizedDescription)")
        }
    }
}
```
**Challenge Solved**: Managing 8 different audio samples with low-latency playback for realistic piano experience.

### **3. Music Learning System**
The app features a complete music education system:

#### **Music Sheet Rendering (`MusicSheetView.swift`)**
```swift
struct MusicSheetView: View {
    let notes: [MusicNote]
    @Binding var highlightedIndex: Int
    
    var body: some View {
        VStack(spacing: 30) {
            // Render multiple staves for long songs
            ForEach(0..<3) { section in
                ZStack(alignment: .topLeading) {
                    StaffView(lines: 1, space: 2)
                    ForEach(notesInSection(section)) { note in
                        NoteView(note: note, currentNote: isHighlighted(note))
                    }
                }
            }
        }
    }
}
```

#### **Real-time Note Feedback**
```swift
// Piano.swift - Interactive piano with learning feedback
struct Piano : View {
    @Binding var highlightedIndex: Int
    let musicNotes: [MusicNote]
    
    var body: some View {
        HStack(spacing: 20) {
            ForEach(notes, id: \.id) { note in
                Model3D(named: musicNotes[highlightedIndex].pitch == note.name ? 
                        "blueNote" : note.modelName, 
                        bundle: realityKitContentBundle)
                    .onTapGesture {
                        SoundManager.shared.playSound(named: note.name)
                        // Progressive learning logic
                        if musicNotes[highlightedIndex].pitch == note.name {
                            highlightedIndex += 1  // Move to next note
                        }
                    }
            }
        }
    }
}
```

### **4. State Management Flow**
The app uses a guided learning flow with multiple states:

```swift
enum AppState {
    case welcome, songInfo, tutorial, experienceMode, piano
}

struct ContentView: View {
    @State private var appState: AppState = .welcome
    
    var body: some View {
        ZStack {
            VStack {
                switch appState {
                case .welcome:
                    WelcomeView { appState = .songInfo }
                case .songInfo:
                    SongInfoView { appState = .tutorial }
                case .tutorial:
                    TutorialView { appState = .experienceMode }
                case .experienceMode:
                    ExperienceModeView { isImmersive in
                        appState = .piano
                    }
                case .piano:
                    PianoOverlayView()
                }
            }
        }
    }
}
```

## 🚀 Getting Started

### **Prerequisites**
- Apple Vision Pro or VisionOS simulator
- Xcode 15.0 or higher
- iOS 17.0+ / visionOS 1.0+

### **Installation**
```bash
# 1. Clone the repository
git clone https://github.com/yocelynnns/echo-hackathon.git

# 2. Open in Xcode
open echo-Hackathon/immersion.xcodeproj

# 3. Select VisionOS as target
# 4. Build and run on Vision Pro or simulator
```

## 🎮 How to Use

### **Learning Flow**
1. **Welcome Screen**: Introduces the app experience
2. **Song Selection**: Choose "Twinkle Twinkle Little Star" as first song
3. **Interactive Tutorial**: Learn finger positioning and music basics
4. **Mode Selection**: Choose between immersive or overlay mode
5. **Playing Experience**: Follow highlighted notes on the virtual piano

### **Technical Interaction**
- **Hand Tracking**: Uses Vision Pro's built-in sensors to detect finger positions
- **Collision Detection**: Physics-based interaction between fingertips and virtual keys
- **Audio-Visual Sync**: Perfect timing between key press and sound playback
- **Progressive Difficulty**: Automatically adjusts to user's skill level

## 🔮 Technical Challenges & Solutions

### **Challenge 1: Low-latency Audio Playback**
**Problem**: AR experiences require <20ms latency for realistic feedback.
**Solution**: Pre-loaded audio buffers and optimized AVAudioPlayer configuration.

### **Challenge 2: Accurate Hand-to-Key Mapping**
**Problem**: Mapping 3D finger positions to 2D piano keys.
**Solution**: Collision-based detection with adjustable sensitivity zones.

### **Challenge 3: Music Learning Algorithm**
**Problem**: Adaptive difficulty based on user performance.
**Solution**: State machine that tracks correct/incorrect notes and adjusts tempo.

## 🎓 Learning Outcomes

### **Technical Skills Developed**
- **Spatial Computing**: VisionOS development and RealityKit integration
- **Hand Tracking Optimization**: Selective joint tracking for performance
- **3D Audio Spatialization**: Creating immersive sound environments
- **SwiftUI + RealityKit**: Bridging 2D UI with 3D AR content
- **Gesture Recognition**: Natural user interface design

## 🏆 Hackathon Achievement
**Echo** was developed during a 48-hour hackathon, demonstrating:
- Rapid prototyping of complex AR applications
- Innovative use of Apple's latest spatial computing platform
- Integration of multiple technologies (AR, audio, hand tracking)
- User-centered design for educational applications

## 📺 Demo & Presentation
### 🎥 YouTube Demo
https://youtu.be/aNHaNLUb9bs

### 📊 Presentation Slides
Add your PowerPoint or slide deck link here:
<insert PowerPoint link>

## 🔮 Future Enhancements

### **Planned Features**
1. **Song Library**: Expandable collection with popular tunes
2. **Machine Learning**: Adaptive difficulty based on playing style
3. **Progress Bar**: A visual meter that fills as the player moves through a song, showing how much is completed and how much remains.

### **Technical Improvements**
- **ML-based Gesture Recognition**: More accurate finger tracking
- **Cloud Sync**: Progress tracking across devices

## 👥 Team & Credits
Developed by  
[@novailable](https://github.com/novailable),  
[@nann5an1](https://github.com/nann5an1),  
[@thanthtetaung4](https://github.com/thanthtetaung4),  
[@yocelynnns](https://github.com/yocelynnns)  
as a collaborative exploration of spatial computing’s potential for music education.

---

**🎹 Transforming musical education through spatial computing.**
