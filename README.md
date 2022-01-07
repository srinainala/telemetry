package com.aa.baggage.rampopslogmanager;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;
import androidx.lifecycle.Observer;
import androidx.work.WorkInfo;
import androidx.work.WorkManager;

import android.Manifest;
import android.app.ActivityManager;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.BatteryManager;
import android.os.Build;
import android.os.Bundle;
import android.telephony.CellInfo;
import android.telephony.CellInfoLte;
import android.telephony.CellSignalStrengthLte;
import android.telephony.TelephonyManager;
import android.util.Log;
import android.view.View;
import android.widget.TextView;

import com.aa.baggage.rampopslogmanager.uploadworkmanager.LogUploadManager;
import com.aa.baggage.rampopslogmanager.utility.RampLogger;

import java.util.List;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = MainActivity.class.getName();
    TextView lblStatus;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        lblStatus = findViewById(R.id.lblStatus);
        setTitle("RampOps Log Manager");

        WorkManager.getInstance(this).getWorkInfosByTagLiveData(getString(R.string.upload_log_worker_tag))
                .observe(this, new Observer<List<WorkInfo>>() {
                    @Override
                    public void onChanged(List<WorkInfo> workInfos) {
                        if (workInfos != null && workInfos.size() > 0) {
                            RampLogger.d(TAG, "Log upload WorkManager run attempt count: " +workInfos.get(0).getRunAttemptCount());
                            String workManagerStatus =
                                    String.format("Log upload WorkManager Status: %s", workInfos.get(0).getState().toString());
                            lblStatus.setText(workManagerStatus);
                            RampLogger.d(TAG, workManagerStatus);
                        }

                    }
                });
        registerBatteryReceiver();
        memoryInfo();
    }

    @Override
    public void onStart() {
        super.onStart();
        startLogUploadManager();
        //finish();
    }

    public void startLogUploadManager(View view) {
        startLogUploadManager();
    }

    private void startLogUploadManager() {
        int iLogUploadInterval = getApplicationContext()
                .getResources().getInteger(R.integer.default_logupload_interval);
        LogUploadManager.startLogUploadManger(getApplicationContext(), iLogUploadInterval);
    }

    //New Code added from here
    private void printSignalStrength() {

        Log.d("DeviceStatus","Signal strength ");

        TelephonyManager telephonyManager = (TelephonyManager) getSystemService(AppCompatActivity.TELEPHONY_SERVICE);

        //val telephonyManager = (TELEPHONY_SERVICE) as TelephonyManager

        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
        ) != PackageManager.PERMISSION_GRANTED
        ) {
            String[] permissions = {Manifest.permission.ACCESS_FINE_LOCATION};
            ActivityCompat.requestPermissions(this, permissions, 10001);
            return;
        } else {
            TelephonyManager tels = (TelephonyManager) getSystemService(AppCompatActivity.TELEPHONY_SERVICE);
            String networkOperator = tels.getNetworkOperator();
            CellInfoLte cellInfoLte = (CellInfoLte) telephonyManager.getAllCellInfo().get(0);
            CellSignalStrengthLte cellSignalStrengthLte = cellInfoLte.getCellSignalStrength();
            //val tower = cellSignalStrengthLte.dbm
            int tower = cellSignalStrengthLte.getAsuLevel();
            Log.d("DeviceStatus", "Signal strength dbm ="+cellSignalStrengthLte.getDbm());
            Log.d("DeviceStatus", "Signal strength asuLevel ="+cellSignalStrengthLte.getAsuLevel());
            Log.d("DeviceStatus", "Signal networkOperator ="+networkOperator);

            RampLogger.d(TAG,"Signal strength dbm ="+cellSignalStrengthLte.getDbm());
            RampLogger.d(TAG,"Signal strength asuLevel ="+cellSignalStrengthLte.getAsuLevel());
            RampLogger.d(TAG,"Signal networkOperator ="+networkOperator);
        }
    }

    //This need to be tested on CDMA Network only
    public void cdmaSignalStatus() {
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
        ) != PackageManager.PERMISSION_GRANTED
        ) {
            String[] permissions = {Manifest.permission.ACCESS_FINE_LOCATION};
            ActivityCompat.requestPermissions(this, permissions, 10001);
        } else {

            TelephonyManager telephonyManager = (TelephonyManager)
                    getSystemService(AppCompatActivity.TELEPHONY_SERVICE);

            String networkOperator = telephonyManager.getNetworkOperator();
//            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
//                CellInfo cellSignalStrengthLte = telephonyManager.getAllCellInfo().get(0).get.getCellSignalStrength();
//
//
//                Log.d("DeviceStatus", "Signal strength dbm:  $cellSignalStrengthLte.dbm ")
//                Log.d("DeviceStatus", "Signal strength asuLevel:  ${cellSignalStrengthLte.asuLevel} ")
//                Log.d("DeviceStatus", "Signal networkOperator: $networkOperator")
//                binding.signalStatusTv.text = "Signal strength: $cellSignalStrengthLte.asuLevel "
//
//            } else {
//                TODO("VERSION.SDK_INT < R")
//            }
        }
    }

    private void registerBatteryReceiver() {
        registerReceiver(batteryReceiver,new IntentFilter(Intent.ACTION_BATTERY_CHANGED));

    }

    private BroadcastReceiver batteryReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {

            int level = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);
            int scale = intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1);

            float batterLevel = level * 100 / (float) scale;

            Log.d("DeviceStatus", "battery percentage:"+batterLevel);
            RampLogger.d(TAG,"battery percentage:"+batterLevel);
        }
    };

    public void checkForTelephonyPermission() {

        if(ContextCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_FINE_LOCATION
            ) == PackageManager.PERMISSION_GRANTED) {
                printSignalStrength();
            }
            else {
            String[] permissions = {Manifest.permission.ACCESS_FINE_LOCATION};

            ActivityCompat.requestPermissions(this, permissions, 10001);
            }
        }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if(requestCode == 10001 && grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            {
                printSignalStrength();
            }
        }
    }

    private void showMessage() {
        new AlertDialog.Builder(this)
                .setTitle("Location access permission required")
                .setMessage("Are you sure you want to delete this entry?")
                .setPositiveButton(android.R.string.yes, new DialogInterface.OnClickListener() {
                    public void onClick(DialogInterface dialog, int which) {
                        // Continue with delete operation
                    }
                })
                .setNegativeButton(android.R.string.no, null)
                .setIcon(android.R.drawable.ic_dialog_alert)
                .show();
    }

    private void memoryInfo() {
        ActivityManager activityManger = (ActivityManager) this.getSystemService(ACTIVITY_SERVICE);
        ActivityManager.MemoryInfo memInfo = new ActivityManager.MemoryInfo();// =  (ActivityManager.MemoryInfo) activityManger.getMemoryInfo();
        activityManger.getMemoryInfo(memInfo);

        Log.d("DeviceStatus", "Available Memory:"+memInfo.availMem);
        Log.d("DeviceStatus", "Available Memory:"+memInfo.totalMem);
        Log.d("DeviceStatus", "Available Memory:"+memInfo.threshold);

        RampLogger.d(TAG, "Available Memory:"+memInfo.availMem);
        RampLogger.d(TAG,"Available Memory:"+memInfo.totalMem);
        RampLogger.d(TAG, "Available Memory:"+memInfo.threshold);
    }
}
