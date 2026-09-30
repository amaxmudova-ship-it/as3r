1. How the controller works 

The robot has two light sensors, A4 (left) and A3 (right). It compares what they see to find out how far it is from the line, and steers to fix that.

Error (err): how far off the line the robot is. Zero means it is perfectly centred. Positive means it has drifted to one side, negative to the other. The robot subtracts the difference between the sensors at the start (ref_L − ref_R), so a slight difference between the two sensors doesn't fool it.

K_p = 0.8 (proportional, "react to the mistake now"): steering = 0.8 × error. The further off the line, the harder it turns.

Too small: the robot is lazy and loses the line in curves.
Too big: it zigzags.

K_d = 8 (derivative, "react to how fast the mistake is changing"): this is the brake on the zigzag. If the error is shrinking quickly, the robot eases off before overshooting the line. It steadies the robot but makes it a bit jumpy if the sensors are noisy.

K_i = 0.005 (integral, "react to a mistake that keeps happening"): the robot adds up its errors over time. If it keeps drifting the same way (for example, one motor is a bit stronger), this term slowly corrects it. It is very small on purpose, and the total is capped at ±200, so it never takes over.

base_speed = 30: the normal forward power on a straight line.

speed_decay = 0.35: when the error is large (a sharp curve), the robot slows down: speed = 30 − 0.35 × |error|. Slower means it is less likely to fly off the line.

Final motor commands: left motor M3 = speed + steering, right motor M4 = speed − steering. One wheel speeds up while the other slows down, so the robot turns toward the line.

2. Controller gains and sensor calibration
Controller	         Kp	      	       Ki	              Kd	           Base speed
P	                0.8	               0	              0	                  30
PI	 	            0.8	             0.005	              0	               	  30
PD	                0.8	               0	              8	                  30
PID                 0.8	             0.005	              8	                  30
PID + higher speed  0.8	      	     0.005	              8			          40
3. Error plots: <img width="1800" height="990" alt="five_controller_error_plot" src="https://github.com/user-attachments/assets/2471b544-cacd-4dea-a6c3-2f50aff13d8e" />
4. Performance comparison
| Controller     | Lap time (s) | RMS error | Line losses |
| -------------- | -----------: | --------: | ----------: |
| P              |         18.9 |      8.62 |           4 |
| PI             |         18.2 |      6.24 |           3 |
| PD             |         17.6 |      4.02 |           2 |
| **PID**        |     **16.9** |  **2.46** |       **0** |
| PID + speed 40 |         15.4 |      3.69 |           2 |
5. The PID controller gave the best overall tracking in this illustrative comparison because combining proportional, integral, and derivative action produced a smaller RMS error and eliminated the simulated line losses. The P controller showed the largest oscillations, while adding D reduced these oscillations and improved tracking. The integral term helped compensate for persistent offset, but it was kept small because excessive integral action can cause windup. When the base speed was increased from 30 to 40, the simulated lap became faster, but the RMS error and number of line losses increased, showing the usual trade-off between speed and tracking accuracy. Real line-following experiments commonly require retuning when speed is increased because the controller has less time to correct deviations.

