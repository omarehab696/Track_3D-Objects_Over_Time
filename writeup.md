# Tracking Result
The tracking result of one scene is in same directory as as a video.


# Writeup: Track 3D-Objects Over Time

Please use this starter template to answer the following questions:

### 1. Write a short recap of the four tracking steps and what you implemented there (filter, track management, association, camera fusion). Which results did you achieve? Which part of the project was most difficult for you to complete, and why?

During the EKF (Extended Kalman Filter) step, I utilized the Kalman Filter to predict the state of objects, which includes their position and velocity in 3D space, and then updated their state from sensor measurements. The sensor could be a camera, LiDAR, or Radar. To execute the Kalman filter, I implemented the predict and update methods, as well as related methods like get_hx and get_H. This allowed me to predict the object state, which consists of the mean and variance of its state, and update its state with sensor measurements. Converting the coordinates from vehicle to sensor coordinates was necessary to update the state with measurements, which is why the get_hx or get_H method was required.

The original Kalman filter assumes that the system is linear. However, to convert the state coordinate from vehicle to camera sensor, a non-linear transformation is required. Therefore, I used the EKF, which approximates non-linear systems as linear systems using a Taylor series expansion.

In the track management step, a new track is initialized with its state ("initialized") and score, and then tracked based on the EKF result as the ego vehicle moves. The track score should decrease if the track is not detected in the last several frames, and the track should be deleted if its score is below a threshold or its variance is larger than a variance threshold. The track state needs to remain "confirmed" if the track remains detected by sensors.

To manage tracks, I first implemented the manage_tracks method to decrease the score for unassigned tracks and then decided on score and variance thresholds to delete old and useless tracks. In addition, by the handle_updated_track method, I could increase the track score and set the track state to "tentative" or "confirmed" based on my own threshold.

In the association step, a matching process between existing tracks and new measurements from sensors is done. I used SNN (Single Nearest Association) to associate tracks and measurements and calculate the distance between them using the Mahalanobis distance, which can reflect not only the actual distance but also the variance of tracks.

In the associate method, I implemented an association_matrix between tracks and measurements by calculating the Mahalanobis distance and gating the irrelevant matchings. Afterward, the closest track and measurement are returned by the get_closest_track_and_meas method, and then the update method is executed.

Finally, in the camera fusion step, I had to implement the non-linear system since the transformation from 3D space (the real world) to 2D image space is non-linear. To do this, I wrote the get_hx method for the camera transformation.

After implementing all of the codes above, I was able to initialize new tracks from sensor detections, track them by their track score and state, and decide whether to keep them on the tracking list or delete them and not track them.

The most challenging part of this project was understanding the entire object tracking process. To overcome this, I structured each element, such as Trackmanagement and Association, by writing them down on a large paper and looking at the big picture. By doing so, I was able to clearly understand the entire system, making it easier to implement each element.

### 2. Do you see any benefits in camera-lidar fusion compared to lidar-only tracking (in theory and in your concrete results)? 

In my opinion, camera-lidar fusion offers several advantages over lidar-only tracking. Firstly, during my project, I noticed a significant reduction in the number of ghost tracks when I incorporated camera measurements into the tracking system, which was not the case with lidar-only tracking. Moreover, by using camera-lidar fusion, the probability of a false negative case, where the sensor fails to detect an object - which could be potentially critical - is decreased. This is because objects that are difficult to detect by lidar can be easily detected by the camera.

### 3. Which challenges will a sensor fusion system face in real-life scenarios? Did you see any of these challenges in the project?

From my perspective, encountering a false negative case is one of the most challenging and hazardous scenarios in real-life situations. Although using a sensor fusion system that incorporates multiple sensors, such as camera, radar, and lidar, to complement each other's shortcomings, reduces the likelihood of false negative cases, it is not guaranteed to be zero. Moreover, the Association step can pose difficulties since heuristic methods are commonly employed for distance measurements, which may not always be accurate.

The process of associating measurements with tracks can be arduous, especially in a scenario with numerous vehicles or objects to track, where the heuristics employed to assign measurements to tracks may fail. Furthermore, in situations where either one of the sensors fails to detect an object in the scene, the tracking of that object may never be confirmed. Additionally, some sensors might perform better or worse in specific situations. For instance, cameras may not perform well at night, while lidar may not perform well in adverse weather conditions. Although we did not experience any of these specific scenarios in our project, it is conceivable that such situations may arise in real-world circumstances.


### 4. Can you think of ways to improve your tracking results in the future?

In my new role in V2X communication, I've discovered that V2X messages can offer a promising approach to enhancing tracking results. By exchanging information with other vehicles or infrastructure through V2V or V2I messages, we can supplement our tracking information and improve its accuracy. Additionally, better detection results would also lead to improved tracking. If the detection results are reliable, we can place more trust in the measurements rather than relying on predictions, which are less accurate.
