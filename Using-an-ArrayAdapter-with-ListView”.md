<?xml version="1.0" encoding="utf-8"?>
<ListView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/listItem"
    android:orientation="vertical"
    android:padding="10dp"
    android:numColumns="2"
    tools:context=".NumberActivity">

</ListView>

package com.example.android.trnslate;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.GridLayout;
import android.widget.GridView;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.TextView;

import java.util.ArrayList;

public class NumberActivity extends AppCompatActivity {

    ArrayList<String> wordss= new ArrayList<String>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_number);
        addToArry();

        ArrayAdapter<String> itemAdapter = new ArrayAdapter<>(this,android.R.layout.simple_list_item_1,wordss);
        ListView listView =findViewById(R.id.listItem);
       // GridView listView = findViewById(R.id.listItem);

        listView.setAdapter(itemAdapter);


    }

    private void addToArry() {
        wordss.add("one");
        wordss.add("Tow");
        wordss.add("Three");
        wordss.add("Fowr");
        wordss.add("Five");
        wordss.add("Sex");
        wordss.add("Seven");
        wordss.add("Eight");
        wordss.add("nine");
        wordss.add("Ten");

    }
}
