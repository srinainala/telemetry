package com.example.deviceutils

import android.Manifest
import android.content.DialogInterface
import android.content.Intent
import android.content.IntentFilter
import android.content.pm.PackageManager
import android.os.BatteryManager
import android.os.Bundle
import android.telephony.CellInfoLte
import android.telephony.TelephonyManager
import android.util.Log
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import androidx.core.content.ContextCompat.getSystemService
import androidx.navigation.fragment.findNavController
import com.example.deviceutils.databinding.FragmentFirstBinding
import android.R
import android.app.ActivityManager
import android.content.Context.ACTIVITY_SERVICE
import android.os.Build

import android.widget.TextView

import android.telephony.CellSignalStrengthGsm

import android.telephony.CellInfoGsm

import androidx.core.content.ContextCompat.getSystemService




/**
 * A simple [Fragment] subclass as the default destination in the navigation.
 */
class FirstFragment : Fragment() {

    private var _binding: FragmentFirstBinding? = null

    // This property is only valid between onCreateView and
    // onDestroyView.
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {

        _binding = FragmentFirstBinding.inflate(inflater, container, false)
        return binding.root

    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        checkForTelephonyPermission()
        printBatteryStatus()
        memoryInfo()
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }

    fun printSignalStrength() {

        Log.d("DeviceStatus","Signal strength ")

        val telephonyManager = activity?.getSystemService(AppCompatActivity.TELEPHONY_SERVICE) as TelephonyManager

        //val telephonyManager = (TELEPHONY_SERVICE) as TelephonyManager

        if (ActivityCompat.checkSelfPermission(
                requireActivity(),
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(requireActivity(), arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), 10001)
            return
        } else {


            val tels = requireActivity().getSystemService(AppCompatActivity.TELEPHONY_SERVICE) as TelephonyManager
            val networkOperator = tels.networkOperator
            val cellInfoLte = telephonyManager.allCellInfo[0] as CellInfoLte
            val cellSignalStrengthLte = cellInfoLte.cellSignalStrength
            //val tower = cellSignalStrengthLte.dbm
            val tower = cellSignalStrengthLte.asuLevel
            Log.d("DeviceStatus", "Signal strength dbm = $cellSignalStrengthLte.dbm ")
            Log.d("DeviceStatus", "Signal strength asuLevel = ${cellSignalStrengthLte.asuLevel} ")
            Log.d("DeviceStatus", "Signal networkOperator = $networkOperator")
            binding.signalStatusTv.text = "Signal strength = $tower "

        }
    }

    //This need to be tested on CDMA Network only
    fun cdmaSignalStatus() {
        if (ActivityCompat.checkSelfPermission(
                requireActivity(),
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(requireActivity(), arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), 10001)
            return
        } else {

            val telephonyManager =
                requireActivity().getSystemService(AppCompatActivity.TELEPHONY_SERVICE) as TelephonyManager

            val networkOperator = telephonyManager.networkOperator
            val cellSignalStrengthGsm = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
                val cellSignalStrengthLte = telephonyManager.allCellInfo.get(0).cellSignalStrength


                Log.d("DeviceStatus", "Signal strength dbm:  $cellSignalStrengthLte.dbm ")
                Log.d("DeviceStatus", "Signal strength asuLevel:  ${cellSignalStrengthLte.asuLevel} ")
                Log.d("DeviceStatus", "Signal networkOperator: $networkOperator")
                binding.signalStatusTv.text = "Signal strength: $cellSignalStrengthLte.asuLevel "

            } else {
                TODO("VERSION.SDK_INT < R")
            }
        }
    }

    private fun printBatteryStatus() {
        Log.d("DeviceStatus","battery status")

        val batteryStatus: Intent? = IntentFilter(Intent.ACTION_BATTERY_CHANGED).let { filter ->
            requireActivity().applicationContext.registerReceiver(null, filter)
        }

        val batteryPct: Float? = batteryStatus?.let { intent ->
            val level: Int = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1)
            val scale: Int = intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
            level * 100 / scale.toFloat()
        }

        Log.d("DeviceStatus", "battery percentage: $batteryPct")
        binding.batterStatusTv.text = "Battery percentage: $batteryPct"
    }

    fun checkForTelephonyPermission() {
        when (PackageManager.PERMISSION_GRANTED) {
            ContextCompat.checkSelfPermission(
                requireActivity(),
                Manifest.permission.ACCESS_FINE_LOCATION
            ) -> {
                printSignalStrength()
            }
            else -> {

                ActivityCompat.requestPermissions(requireActivity(), arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), 10001)
            }
        }
    }

    override fun onRequestPermissionsResult(requestCode: Int,
                                            permissions: Array<String>, grantResults: IntArray) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when (requestCode) {
            10001 -> {
                // If request is cancelled, the result arrays are empty.
                if ((grantResults.isNotEmpty() &&
                            grantResults[0] == PackageManager.PERMISSION_GRANTED)) {
                    // Permission is granted. Continue the action or workflow
                    // in your app.
                    printSignalStrength()
                } else {
                    showMessage()
                }
                return
            }
        }
    }

    private fun showMessage() {
        val alertDialog: AlertDialog? = activity?.let {
            val builder = AlertDialog.Builder(it)
            builder.apply {
                setPositiveButton("Ok",
                    DialogInterface.OnClickListener { dialog, id ->
                        // User clicked OK button
                    })
                setNegativeButton("Cancel",
                    DialogInterface.OnClickListener { dialog, id ->
                        // User cancelled the dialog
                    })
            }

            // Create the AlertDialog
            builder.create()
        }
    }

    fun memoryInfo() {
        val activityManger = requireActivity().getSystemService(ACTIVITY_SERVICE) as ActivityManager
        var memInfo =  ActivityManager.MemoryInfo()
        activityManger.getMemoryInfo(memInfo)
        memInfo.availMem
        memInfo.totalMem
        memInfo.threshold

        Log.d("DeviceStatus", "Available Memory: ${memInfo.availMem}")
        Log.d("DeviceStatus", "Available Memory: ${memInfo.totalMem}")
        Log.d("DeviceStatus", "Available Memory: ${memInfo.threshold}")

        binding.memAvailableTv.text = "Available Memory: ${memInfo.availMem}"
        binding.memoryTotalTv.text = "Total Memory: ${memInfo.totalMem}"

    }
}
