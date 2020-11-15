---
layout: post
title: Using Text-to-Speech and Speech-to-Text in Android
author: Daniel Müller
date: '2020-11-15'
category: guides
tags: android
summary: Using Text-to-Speech and Speech-to-Text in Android
thumbnail: post3_android_tts_stt/logo.jpg
---

# Using Text-to-Speech and Speech-to-Text in Android

Logo Information: [Designed by pch.vector / Freepik](http://www.freepik.com)

I'm writing an Android app, which should be able to:

* Reading a text, that was typed into a text-field (=> Text-to-Speech)
* Writing a recognized text from user's speach into a text-field (=> Speech-to-Text)

I want to use this blog post for taking some notes for myself and showing my example code to you, happy coding :-)

### Creating a new Android project

I'm using the Android Studio IDE for writing my apps. Here I created a new project with the name "SpeechTextExample". I created the project with an "empty activity" and the "Java" programming language with minimum Android SDK 16 (Android 4.1).

### Defining the layout

The example app will only contain one activity, so it will also have just one layout. In this layout, I want to have the following elements:

**Text-To-Speech sector**

* EditText (input text-field) for giving a text, which should be read
* Spinner (dropdown) for selecting the corresponding language, in which the text is written and read
* Button for starting the text reading

**Speech-to-Text sector**

* Spinner for language selection
* Button for starting the voice recording and "translation" into text
* TextField for showing the identified text, that was spoken before

**activity_main.xml**

Here is the code of my MainActivity's layout, it defines the elements listed above:

``````xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Text-To-Speech"
        android:textAppearance="@style/TextAppearance.AppCompat.Medium"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Spinner
        android:id="@+id/spinnerLanguageTextToSpeech"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:spinnerMode="dropdown"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView" />

    <EditText
        android:id="@+id/txtTextToSpeech"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:ems="10"
        android:inputType="textPersonName"
        android:text="Enter your Text here."
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/spinnerLanguageTextToSpeech" />

    <Button
        android:id="@+id/btnTextToSpeech"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:onClick="btnTextToSpeechClicked"
        android:text="Read the text"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/txtTextToSpeech" />

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:text="Speech-To-Text"
        android:textAppearance="@style/TextAppearance.AppCompat.Medium"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/btnTextToSpeech" />

    <Spinner
        android:id="@+id/spinnerLanguageSpeechToText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:spinnerMode="dropdown"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView2" />

    <Button
        android:id="@+id/btnSpeechToText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:onClick="btnSpeechToTextClicked"
        android:text="Record my Speech"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/spinnerLanguageSpeechToText" />

    <EditText
        android:id="@+id/txtSpeechToText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:clickable="false"
        android:editable="false"
        android:ems="10"
        android:inputType="textPersonName"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/btnSpeechToText" />

</androidx.constraintlayout.widget.ConstraintLayout>
``````

### Defining a custom List adapter

The layout defines two Spinners (`spinnerLanguageSpeechToText` and `spinnerLanguageTextToSpeech`) for language selection. We need to fill them with some values. In this case, we want to set a specific language for the Text-to-Speech and Speech-to-Text engines, depending on the selection of the Spinner element. Both engines need a object of class `Locale`, which represents a country code / language combination. For using a list of Locales as our language spinner elements, we need to define a custom adapter. However, here is the code for a basic custom `ArrayAdapter<Locale>`:

``````java
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.Locale;

public class LanguageAdapter extends ArrayAdapter<Locale> {

    public LanguageAdapter(Context context, ArrayList<Locale> locales) {
        // Calling the super constructor with default layout
        super(context, android.R.layout.simple_list_item_1, locales);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        // Get the Locale element on current position
        Locale locale = getItem(position);

        // Create a view (spinner element)
        if (convertView == null) {
            convertView = LayoutInflater.from(getContext()).inflate(android.R.layout.simple_list_item_1, parent, false);
        }

        // The default layout has one TextView with id "text1"
        TextView textView = (TextView) convertView.findViewById(android.R.id.text1);
        // Set a string (country code + language code) for the TextView
        textView.setText(locale.getCountry() + " - " + locale.getLanguage());

        return convertView;
    }
}
``````

### The code of the MainActivity

Finally we need our MainActivity code, which contains the entry point when starting the app (`onCreate` method), the methods that are called when one of the buttons is pressed (`btnTextToSpeechClicked` and `btnSpeechToTextClicked`) and some other helper methods (e.g. `setLanguageSpinners` and `initiateTextToSpeechEngine`).

Hint: The Speech-To-Text engine is started as a new Activity, therefore the `startActivityForResult` method is called. The result of this "external" Activity (where the user can speak a text) is received in the `onActivityResult` method, so we can use this method to evaluate the result of the spoken text.

Here is the code of the `MainActivity.java` file:

``````java
import androidx.appcompat.app.AppCompatActivity;
import android.app.Activity;
import android.content.ActivityNotFoundException;
import android.content.Intent;
import android.os.Bundle;
import android.speech.RecognizerIntent;
import android.speech.tts.TextToSpeech;
import android.view.View;
import android.widget.EditText;
import android.widget.Spinner;
import android.widget.Toast;
import java.util.ArrayList;
import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    // The request code (unique number) for the Speech-To-Text intent
    private static final int REQUEST_CODE_STT = 1;

    // The instance for text-to-speech
    private TextToSpeech textToSpeech;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // Default Android Activity code
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Setup the Language Spinners
        setLanguageSpinners();

        // Initiate the Text-To-Speech Engine
        initiateTextToSpeechEngine();
    }

    private void setLanguageSpinners() {
        // Create a List of all languages (locales), that the app shall support, e.g. English and German
        ArrayList<Locale> languagesToSupport = new ArrayList<Locale>();
        languagesToSupport.add(Locale.US);
        languagesToSupport.add(Locale.GERMANY);

        // Create a custom Spinner adapter for the locale list
        LanguageAdapter languageAdapter = new LanguageAdapter(this, languagesToSupport);

        // Set the Text-to-Speech Language Spinner
        Spinner spinnerLanguageTextToSpeech = findViewById(R.id.spinnerLanguageTextToSpeech);
        spinnerLanguageTextToSpeech.setAdapter(languageAdapter);
        spinnerLanguageTextToSpeech.setSelection(0);

        // Set the Speech-to-Text Language Spinner
        Spinner spinnerLanguageSpeechToText = findViewById(R.id.spinnerLanguageSpeechToText);
        spinnerLanguageSpeechToText.setAdapter(languageAdapter);
        spinnerLanguageSpeechToText.setSelection(0);
    }

    private void initiateTextToSpeechEngine() {
        // Using the TextToSpeech constructor for creating a new text-to-speech engine
        textToSpeech = new TextToSpeech(getApplicationContext(), new TextToSpeech.OnInitListener() {
            @Override
            public void onInit(int status) {
                if (status != TextToSpeech.SUCCESS) {
                    // Print an error message when initialization failed
                    Toast.makeText(getApplicationContext(), "Text-To-Speech: Initialization failed!", Toast.LENGTH_LONG).show();
                }
            }
        });
    }

    public void btnTextToSpeechClicked(View v) {
        // Getting the text for text-to-speech "translation"
        EditText txtTextToSpeech = findViewById(R.id.txtTextToSpeech);
        String textToRead = txtTextToSpeech.getText().toString();

        // Getting the language (locale) for reading the text
        Spinner spinnerTextToSpeech = findViewById(R.id.spinnerLanguageTextToSpeech);
        Locale locale = (Locale)spinnerTextToSpeech.getSelectedItem();

        // Set the current locale for the text-to-speech engine => returns result code
        int localeSettingResult = textToSpeech.setLanguage(locale);

        // Check result code and print toast on error
        if (localeSettingResult == TextToSpeech.LANG_MISSING_DATA || localeSettingResult == TextToSpeech.LANG_NOT_SUPPORTED) {
            Toast.makeText(getApplicationContext(), "Text-To-Speech: Language not supported!", Toast.LENGTH_LONG).show();
            return;
        }

        // Read the given text => returns result code
        int speakingResult = textToSpeech.speak(textToRead, TextToSpeech.QUEUE_FLUSH, null);

        // Check result code and print toast on error
        if (speakingResult == TextToSpeech.ERROR) {
            Toast.makeText(getApplicationContext(), "Text-To-Speech: Error in text conversion!", Toast.LENGTH_LONG).show();
            return;
        }
    }

    public void btnSpeechToTextClicked(View v) {
        // Getting the language (locale) for reading the text
        Spinner spinnerLanguageSpeechToText = findViewById(R.id.spinnerLanguageSpeechToText);
        Locale locale = (Locale)spinnerLanguageSpeechToText.getSelectedItem();

        // Start a new Intent for Voice Input and set the chosen language
        Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, locale.toString());
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, locale.toString());
        intent.putExtra(RecognizerIntent.EXTRA_PROMPT, locale.getDisplayLanguage());

        try {
            // Use the intent for starting a new activity => the result comes back and can be processed in activity's "onActivityResult" method
            startActivityForResult(intent, REQUEST_CODE_STT);
        } catch (ActivityNotFoundException a) {
            // Print toast on error
            Toast.makeText(this, "Speech-To-Text: Your device doesn't support this!", Toast.LENGTH_LONG).show();
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        // This is the callback method, that is called, when the intent for speech-to-text request comes back
        super.onActivityResult(requestCode, resultCode, data);

        // Check for the correct request code (this is important, when different intents are used (not in this example))
        if(requestCode == REQUEST_CODE_STT) {

            // Check the response code of the result => Write the recognized text (success) OR print error message (failure)
            if (resultCode == Activity.RESULT_OK && data != null) {
                String speechToText = data.getStringArrayListExtra(RecognizerIntent.EXTRA_RESULTS).get(0);
                EditText txtSpeechToText = findViewById(R.id.txtSpeechToText);
                txtSpeechToText.setText(speechToText);
            } else {
                Toast.makeText(this, "Speech-To-Text: Something went wrong!", Toast.LENGTH_LONG).show();
            }
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        // Shutdown the text-to-speech instance when closing the activity
        if (textToSpeech != null) {
            textToSpeech.stop();
            textToSpeech.shutdown();
        }
    }
}
``````

