# Standby Dynamic Notch and Dynamic Notch Mute

The Dynamic Notch is very effective at squashing noise peaks, but always contributes the same amount and type of filtering (per the dynamic notch settings) regardless of the amount of peak gyro noise present during a gyro loop iteration (thus providing less benefit during periods of less noise, while still adding filter latency).

Standby Dynamic Notch and Mute are two Dynamic Notch options (usable together) that allow for different dynamic notch behavior to be triggered during periods of reduced gyro noise on a given axis.  Both are triggered by comparing their respective configured "trigger" thresholds to the value in the FFT 'bin' that the dynamic notch code selected as the noise peak (in other words: how "loud" the noise is around the noisiest frequency); if the observed value is **less** than the trigger setting for the respective options, then they are activated for that gyro loop iteration.

The Standby Dynamic Notch allows an alternative "standby" notch configuration to be used on an axis while it is triggered.  An alternate Q and width value are specified, typically to configure a "narrower" notch than the primary Dynamic Notch in order to reduce filter latency.  Optionally, the the Standy notch can be set to a static frequency in order to strategically target specific resonance frequencies when high amplitude noise spikes are not present.

The Dynamic Notch Mute causes the Dynamic Notch to not be applid to the gyro signal on an axis while the mute is triggered, thus providing no filtering, but also not contributing to filter latency. When the Dynamic Notch Mute is triggered, neither the normal Dynamic Notch or the Standby Dynamic notch filter will be applied to the gyro signal.

Note that both the Dynamic Notch and the Standby Dynamic Notch (when enabled with a non-zero trigger value) are maintained even when they are not being applied to the gyro signal, so setting high trigger values is not an effective means of reducing flight controller CPU load.


### Dynamic Notch Mute Configuration Approach
1. Log a flight (with a filtering scheme including the dynamic notch) with debug_mode FFT_BIN_MAX.  The debug [0-3] values represent the magnitude of center 'bin' from Dynamic Notch's FFT calculation that it uses to locate the notch on the roll/pitch/yaw axises.
2. Plot (or perhaps calculate the mathmatical mean / stdev) the debug values.  If there is a significant percentage of entries in a 'low' range (perhaps 0-500), try setting dyn_notch_mute_trigger to a value just above the high end of that range. This will 'mute' (temporarily turn off) the dynamic notch when the signal in the frequency range where the dynamic notch would go is relatively low, thus reducing frequency/phase delay.
3. Log some flights with DYN_STDBY_MUTE_TRIG or DYN_STDBY_MUTE enabled and evaluate the result.  Look for how often the Mute is actually kicking in, the gyro signal during the periods that it is, and the overall effect on PlasmaTree/PidTool gyro noise graphs.
4. Try slowly pushing up the dyn_notch_mute_trigger to further reduce latency as long as the resulting filtering is still acceptable. 

### Alternate Dynamic Notch Mute Approach
- If filtering is generally adequate without Dynamic Notch (such as with effective rpm filtering), set the dyn_notch_mute_trigger to a higher value that would typically not get triggered under normal circumstances.  The Dynamic Notch can then serve as an 'emergency standby' in case the quad sustains damage during flight resulting in new vibrations or resonances.

### Standby Dynamic Notch Configuration Approach
- The Standby Dynamic Notch provides a means to further tailor the Dynamic Notch when less peak noise is present. Lower Dynamic Notch latency can be achieved by making the Standby Dynamic Notch 'narrower' (higher dyn_notch_stdby_q / lower dyn_notch_stdby_width_percent). If there is a consistant resonance frequency, dyn_notch_stdby_static_hz can be set to that frequency to lock the Standby Dynamic Notch to that frequence when it is active. This may also help reduce artifacts in the gyro signal caused by the Dynamic Notch constantly 'jumping around' when there is no clear noise peak.


### New cli vars

**dyn_notch_stdby_q = 200** Allowed range: 1 - 10000

Q of dynamic notch in Standby Dynamic Notch (if enabled).


**dyn_notch_stdby_width_percent = 0** Allowed range: 0-20

Dynamic notch width in Standby Dynamic Notch (if enabled)


**dyn_notch_stdby_static_hz = 0** Allowed range: 0 - 16000

Frequency to (statically) center the Standby Dynamic Notch.  A value of zero causes the Standby Notch to be centered at the same peak frequency targeted by the normal Dynamic Notch.


**dyn_notch_stdby_trigger = 0** Allowed range: 0 - 50000 (zero = disabled)

When the contents of the FFT bin selected as the 'center' by the dynamic notch (multiplied by 100) is less than the value of  dyn_notch_stdby_trigger, the Standby Dynamic Notch filter (as defined by dyn_notch_stdby_q, dyn_notch_stdby_width_percent, and dyn_notch_stdby_static_hz) is applied to the gyro signal instead of the regular Dynamic Notch filter. A value of zero disables this behavior. Note that the Dynamic Notch Mute, if triggered, will mute the Standby Dynamic Notch in addition to the normal dynamic notch. Therefore, dyn_notch_stdby_trigger must be greater than dyn_notch_mute_trigger in order for the Standby Dynamic Notch to ever activate. 


**dyn_notch_mute_trigger = 0** Allowed range: 0 - 50000 (zero = disabled)

When the contents of the FFT bin selected as the 'center' by the dynamic notch (multiplied by 100) is less than the value of dyn_notch_mute_trigger, the dynamic notch filter (or Standby Dynamic Notch filter if triggered) will not be applied.


### New debug modes
**FFT_BIN_MAX**

[0-3] (RPY): Value of the center FFT bin * 100, which is the value compared with the dyn_notch_stdby_trigger and dyn_notch_mute_trigger to determine whether the Standby Dynamic Notch or Dynamic Notch Mute are triggered.

[4] empty


**DYN_STDBY_MUTE_TRIG**

[0-3] (RPY): 0 = Normal Dynamic Notch behavior, 1 = Standby Dynamic Notch triggered, 2 = Dynamic Notch Mute triggered.

[4] empty


**DYN_STDBY_MUTE**

[0] = roll: 0 = Normal Dynamic Notch behavior, 1 = Standby Dynamic Notch triggered, 2 = Dynamic Notch Mute triggered.

[1] = roll: Value of the center FFT bin * 100, which is the value compared with the dyn_notch_stdby_trigger and dyn_notch_mute_trigger to determine whether the Standby Dynamic Notch or Dynamic Notch Mute are triggered.

[2] = roll: center frequency determined by the FFT (regardless of whether Standby or Mute triggered).

[3] = roll: pre-dyn notch gyro data
