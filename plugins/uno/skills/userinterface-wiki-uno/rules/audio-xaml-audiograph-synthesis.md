---
title: AudioGraph for Procedural UI Sounds
impact: LOW
tags: audio, audiograph, synthesis, procedural
---

## AudioGraph for Procedural UI Sounds

For dynamic sounds (click tones that vary by context, completion chimes with different pitches), use AudioGraph instead of pre-recorded files. One AudioGraph instance with AudioFrameInputNode gives full control.

**Incorrect (10 pre-recorded files for pitch variations):**

```
Assets/Audio/tone-c4.wav
Assets/Audio/tone-d4.wav
Assets/Audio/tone-e4.wav
... // 10 files for one interaction
```

**Correct (AudioGraph generates pitches dynamically):**

```csharp
private async Task InitAudioGraph()
{
    var result = await AudioGraph.CreateAsync(
        new AudioGraphSettings(AudioRenderCategory.SoundEffects));
    _graph = result.Graph;
    _outputNode = await _graph.CreateDeviceOutputNodeAsync();
    _inputNode = _graph.CreateFrameInputNode();
    _inputNode.AddOutgoingConnection(_outputNode.DeviceOutputNode);
    _graph.Start();
}

private void PlayTone(float frequency, double durationMs)
{
    // Generate sine wave AudioFrame at frequency
    var frame = GenerateSineFrame(frequency, durationMs, _graph.EncodingProperties);
    _inputNode.AddFrame(frame);
}
```
