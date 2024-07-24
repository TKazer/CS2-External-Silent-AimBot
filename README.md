# CS2-External-Silent-AimBot
Research process of cs2 external silent.
## Analysis and implementation of CS2 external silent aimbot
Nowadays, there is a constant stream of cheats at the CS2, with most of the cheat features already implemented. One interesting aspect that I came across during my research is external silent aim. Of course, silent aim is easily achievable after injecting into the game; you just need to hook the CreateMove function to intercept the CMD structure data and then modify the view angle. However, it is more challenging to implement this method externally, but it is possible to start the research from this function, leading to the analysis process that follows.

## Analysis
### 1. Silent Aim Principle
Since the game is in a 3D world, most directions are expressed using Euler angles, Pitch, Yaw, and Roll. Silent aim involves modifying the shooting rotation angle. By changing the shooting rotation angle while keeping the local camera rotation angle unchanged, you can shoot at a target without aiming at it directly.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/AnglePitc.png)

### 2. Finding the CreateMove Function
First,  using IDA to analyze the "client.dll" file. By directly searching for "CreateMove" in the "String" window, you can find these texts.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/IDA%20String.png)

Select the second one, and by checking the xrefs, you will find only one function.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/IDA%20Function.png)

This function is CreateMove.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/IDA%20Function2.png)

### 3. Dynamic Debugging of the CreateMove Function
The main data for silent aim is the camera's rotation angle data, Rotation{Pitch, Yaw, Roll}. Therefore, first find the address of ViewAngle quickly in the [cs2-dumper] project on GitHub and add it to CE for debugging and locating the register data.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/CE-ViewAngle.png)

Then, jump to the CreateMove function in the Memory Viewer, set a breakpoint, step through the code, and observe the XMM register data in the FPU window to see if you can find relevant rotation angle data. We found that none of the code after this line of assembly code was executed.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/JNG.png)

So skip that section and continue stepping through the code.

After excuting this call, a familiar data appears in the XMM1 register.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/CE-FPU.png)

You can compare it with the rotation angle data and find that this data is the Yaw data of the camera.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/CE-ViewAngle.png)

This call is a bit suspicious. After directly NOP, I fired a shot in the game and found that the direction of the shot bullet was inconsistent with the actual camera direction.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/FireVideo.gif)

It is basically certain that the shooting angle is written inside this call, so mark this function and jump directly to the function for further analysis. (This function is later found to be the function in the first string xrefs call in the previous IDA String list.)

Open the register window and continue to follow the code with F8 single-step. When you follow this line of code, you will find that XMM0 has the first appearance of Pitch data here.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/First%20Pitch.png)

After continuing the execution, you can see that XMM0 and XMM1 are the rotation angles Pitch and Yaw respectively.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/PitchAndYaw.png)

Therefore, it can be determined that the above two key codes obtain the rotation angle data and assign them to [rcx+18] and [rcx+1C] respectively. It can be guessed that the next XMM0 assignment code is the Roll data, but the Roll data is generally 0, so the following part of the code is not discussed here.

### 4. Simple Hook Verification
A simple hook is applied at 0x87C37C offset to verify whether modifying certain memory locations successfully changes the shooting angle. By observing the discrepancy between the shooting direction and the camera direction, you can confirm the effectiveness of the hook.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/Asm.png)

After executing the script, we found that the shooting angle was inconsistent with the camera angle. After modifying [newmem+50] and [newmem+54], we found that the shooting angle of the bullet could be modified correctly. Then we can be sure that this is the feature we need.

![image](https://github.com/TKazer/CS2-External-Silent-AimBot/blob/main/Shooting2.gif)

### 5. Online Game Effect Verification
Testing in a multiplayer room with a friend, the effectiveness of the silent aim feature is confirmed in online spectating and gameplay. The feature affects the shooting direction visible to other players, demonstrating that the functionality is effective online, not just locally.

## Conclusion
In summary, the reverse engineering process for implementing the silent aim feature in CS2 was relatively smooth. While not overly complex, it required good observational skills. This experience was valuable for me, and I welcome any feedback on textual or technical errors in the article for prompt correction. I hope this reverse engineering process serves as inspiration and assistance to others.
