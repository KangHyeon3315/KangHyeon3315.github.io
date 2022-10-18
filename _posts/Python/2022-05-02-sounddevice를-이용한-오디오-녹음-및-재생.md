---
title:  "sounddevice를 이용한 오디오 녹음 및 재생"
excerpt: ""

categories:
    - Python
tags:
    - [sounddevice]

toc: true
toc_sticky: true

date: 2022-05-02
last_modified_at: 2022-05-02


---

이번에는 sounddevice 라이브러리를 이용한 오디오 녹음 및 재생 기능을 구현해보겠습니다.

우선 라이브러리를 설치해보겠습니다.

    pip install sounddevice

우선 첫번째로 녹음 기능입니다.오디오 녹음 방법도 매우 많지만 우선 제가 사용했던 방식인 callback 기능으로 구현해보겠습니다.

    import sounddevice as sd
    
    audio = queue.Queue()
    def recording_callback(indata, frames, time, status):
        audio.put(indata.copy())
    
    // 오디오 스트림 생성
    record_stream = sd.InputStream(device=2, samplerate=self.__samplerate, dtype='int16', channels=1, callback=recording_callback)
    
    record_stream.start() // 녹음 시작
    time.sleep(10)
    record_stream.stop() // 녹음 종료

그리고 오디오 재생 기능은 다음과 같습니다. 위에서 녹음했던 audio데이터를 이용해 for문을 돌며 stream에 write하면 순서대로 재생 할 수 있습니다.

    with sd.Stream(samplerate=rc.samplerate,  dtype='int16', channels=1) as player:
        for ad in audio.queue:
            player.write(ad)

만약 파일로 저장하고 싶으면 다음과 같이 작성하면 됩니다.

    with sf.SoundFile("./temp.wav", mode='w', samplerate=32000, subtype='PCM_16', channels=1) as file:
        for ad in audio.queue:
            file.write(ad)