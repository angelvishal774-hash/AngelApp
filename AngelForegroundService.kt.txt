package com.vishal.angelai

import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.Service
import android.content.Intent
import android.os.Build
import android.os.Bundle
import android.os.Handler
import android.os.IBinder
import android.os.Looper
import android.speech.RecognitionListener
import android.speech.RecognizerIntent
import android.speech.SpeechRecognizer
import androidx.core.app.NotificationCompat

class AngelForegroundService : Service() {

    private var speechRecognizer: SpeechRecognizer? = null

    override fun onCreate() {
        super.onCreate()
        startMyForegroundService()
        initializeSpeechRecognizerSafely()
    }

    private fun startMyForegroundService() {
        val channelId = "angel_ai_channel"
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                channelId,
                "Angel AI Background Service",
                NotificationManager.IMPORTANCE_LOW
            )
            val manager = getSystemService(NotificationManager::class.java)
            manager?.createNotificationChannel(channel)
        }

        val notification: Notification = NotificationCompat.Builder(this, channelId)
            .setContentTitle("Angel AI Active")
            .setContentText("UPI सुरक्षा आणि वॉइस असिस्टंट चालू आहे")
            .setSmallIcon(R.drawable.ic_launcher_foreground)
            .build()

        startForeground(1, notification)
    }

    private fun initializeSpeechRecognizerSafely() {
        Handler(Looper.getMainLooper()).post {
            try {
                if (SpeechRecognizer.isRecognitionAvailable(this)) {
                    speechRecognizer = SpeechRecognizer.createSpeechRecognizer(this)
                    speechRecognizer?.setRecognitionListener(object : RecognitionListener {
                        override fun onReadyForSpeech(params: Bundle?) {}
                        override fun onBeginningOfSpeech() {}
                        override fun onRmsChanged(rmsdB: Float) {}
                        override fun onBufferReceived(buffer: ByteArray?) {}
                        override fun onEndOfSpeech() {}
                        override fun onError(error: Int) {
                            restartListening()
                        }
                        override fun onResults(results: Bundle?) {
                            restartListening()
                        }
                        override fun onPartialResults(partialResults: Bundle?) {}
                        override fun onEvent(eventType: Int, params: Bundle?) {}
                    })
                    startListening()
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }

    private fun startListening() {
        try {
            val intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH).apply {
                putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
                putExtra(RecognizerIntent.EXTRA_LANGUAGE, "mr-IN")
            }
            speechRecognizer?.startListening(intent)
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    private fun restartListening() {
        Handler(Looper.getMainLooper()).postDelayed({
            try {
                speechRecognizer?.cancel()
                startListening()
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }, 1000)
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }

    override fun onDestroy() {
        super.onDestroy()
        try {
            speechRecognizer?.destroy()
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}

