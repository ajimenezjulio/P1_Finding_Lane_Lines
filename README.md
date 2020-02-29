# **Finding Lane Lines on the Road** 

## Writeup Template

**Finding Lane Lines on the Road**
Lane line detection is a fundamental technique for the design of algorithms that allows a car to drive itself. The detection of these lines allows our car to stay on right path and follow it, in the same way that we use our eyes to stay on the lane.

This project uses an approach based on image processing, it uses Python as the main language and OpenCV as complementary framework for image analysis and processing.

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report
---

### 1. Approach

The approach consisted of 6 steps:

1. **Grayscale**:	In order to ease the work a single channel image was used for the processing, OpenCV uses the next method for colorspace conversion: 

	<img src="https://latex.codecogs.com/svg.latex?\text{RGB[A] to Gray:} \quad Y \leftarrow 0.299 \cdot R + 0.587 \cdot G + 0.114 \cdot B" />
	
2. **Gaussian blur**: Blurring is a common technique to reduce noise in images, the gaussian one adds more weight to the pixels in the middle of the kernel, the next function is used for OpenCV to get the gaussian coefficients, a kernel size of 5 was used for this project:

	<img src="https://latex.codecogs.com/svg.latex?\ G_i= \alpha *e^{-(i-( \texttt{ksize} -1)/2)^2/(2* \sigma^2)}"/>  <br>
	<img src="https://latex.codecogs.com/svg.latex?\sigma = 0.3*((ksize-1)*0.5 - 1) + 0.8"/>
	
3. **Canny edge detection** : After blurring a canny edge detection was applied, Canny is a complex technique that work in 4 steps. First a gaussian blurring is computed, then the first derivative in horizontal ($G_x$) and vertical ($G_y$) direction are calculated through the Sobel kernel to find the edge gradient and direction as follows:

	<img src="https://latex.codecogs.com/svg.latex?\ Edge\_Gradient \; (G) = \sqrt{G_x^2 + G_y^2}"/>  <br>
	<img src="https://latex.codecogs.com/svg.latex?\ Angle \; (\theta) = \tan^{-1} \bigg(\frac{G_y}{G_x}\bigg)"/>
	
	Consecutively non-maximum suppression is applied on the image for edge thining and  finally, hysteresis thresholding is applied to suppress undesired edges, for this project the range was defined as $threshold(80 - 200)$.

4. **Mask region of interest** : The mask of the region of interest is an isoceles trapezoid where the lower base is calculated using the width of the image as reference ($b_i = img_w * 2\alpha$), the height takes the height of the image ($img_h * \beta$), and the upper base is calculated taking as reference the image width where its middle point will be the middle point of the image ($b_s = img_w * \lambda$). In this project the values used were $\alpha =0.1$, $\beta = 0.6$, $\lambda = 0.1$.

<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0A%5Cbegin%7Btikzpicture%7D%5Bisosceleles%20trapezium%2F.style%20args%3D%7Bof%20width%20%231%20and%20height%20%232%0Aand%20name%20%233%7D%7Binsert%20path%3D%7B%0A(45%3A%7B%231%2Fsqrt(2)%7D)%20coordinate(%233-TR)%20--%20(-45%3A%7Bsqrt(%232*%232-%231*%231%2F2)%7D)%20coordinate(%233-BR)%20%0A--%20(-135%3A%7Bsqrt(%232*%232-%231*%231%2F2)%7D)%20coordinate(%233-BL)%20--%20(135%3A%7B%231%2Fsqrt(2)%7D)%20coordinate(%233-TL)%20--%20cycle%7D%7D%5D%0A%5Cdraw%5Bisosceleles%20trapezium%3Dof%20width%201%20and%20height%203%20and%20name%20my%20trap%5D%3B%0A%5Cdraw%5Blatex-latex%5D%20(%5Byshift%3D3mm%5Dmy%20trap-TL)%20--%20(%5Byshift%3D3mm%5Dmy%20trap-TR)%0Anode%5Bmidway%2Cfill%3Dwhite%5D%20%7B%24b_s%24%7D%3B%0A%5Cdraw%5Blatex-latex%5D%20(%5Bxshift%3D-2mm%5Dmy%20trap-TL%20-%7C%20my%20trap-BL)%20--%20%0A(%5Bxshift%3D-2mm%5Dmy%20trap-BL)%20node%5Bmidway%2Cfill%3Dwhite%5D%20%7B%24h%24%7D%3B%0A%5Cdraw%5Bdraw%3Dblack%20and%20name%20myRect%5D%20(-2.8%2C%201.7)%20rectangle%20%2B%2B(5.5%2C-3.75)%3B%0A%5Cdraw%5Blatex-latex%5D%20(%5Bxshift%3D-8mm%2C%20yshift%3D-3mm%5D%20my%20trap-BL)%20--%20(%5Byshift%3D-3mm%5D%20my%20trap-BL)%0Anode%5Bmidway%2Cfill%3Dwhite%5D%20%7B%24%5Calpha%24%7D%3B%0A%5Cdraw%5Blatex-latex%5D%20(%5Byshift%3D-3mm%5D%20my%20trap-BL)%20--%20(%5Byshift%3D-3mm%5D%20my%20trap-BR)%0Anode%5Bmidway%2Cfill%3Dwhite%5D%20%7B%24b_i%24%7D%3B%0A%5Cend%7Btikzpicture%7D%0A" alt="
\begin{tikzpicture}[isosceleles trapezium/.style args={of width #1 and height #2
and name #3}{insert path={
(45:{#1/sqrt(2)}) coordinate(#3-TR) -- (-45:{sqrt(#2*#2-#1*#1/2)}) coordinate(#3-BR) 
-- (-135:{sqrt(#2*#2-#1*#1/2)}) coordinate(#3-BL) -- (135:{#1/sqrt(2)}) coordinate(#3-TL) -- cycle}}]
\draw[isosceleles trapezium=of width 1 and height 3 and name my trap];
\draw[latex-latex] ([yshift=3mm]my trap-TL) -- ([yshift=3mm]my trap-TR)
node[midway,fill=white] {$b_s$};
\draw[latex-latex] ([xshift=-2mm]my trap-TL -| my trap-BL) -- 
([xshift=-2mm]my trap-BL) node[midway,fill=white] {$h$};
\draw[draw=black and name myRect] (-2.8, 1.7) rectangle ++(5.5,-3.75);
\draw[latex-latex] ([xshift=-8mm, yshift=-3mm] my trap-BL) -- ([yshift=-3mm] my trap-BL)
node[midway,fill=white] {$\alpha$};
\draw[latex-latex] ([yshift=-3mm] my trap-BL) -- ([yshift=-3mm] my trap-BR)
node[midway,fill=white] {$b_i$};
\end{tikzpicture}
" /></p>

5. **Hough Transform** : Hough transform is a feature extraction method for detecting simple shapes such as circles, lines etc in an image. The line general equation ($y= mx + b$) presents the problem that $m$ can take values between $-\infty$ to $\infty$, so for bounding to a finite state the polar system is used changing the equation into $\rho = x cos(\theta) + y sin(\theta)$.

	Next step is to initialize a new 2D plane called accumulator where we will trace the function of every pixel while varying the value of $\theta$, the intersection points correspond to lines in the Cartesian plane, a vocation occurs to ignore lines that don't met criteria. The next parameters were used in this project:

	$$
	  \rho = 2 
	  \;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\; 
	  \theta = \pi/2 
	  \;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;  
	  min_{votes} = 20
	  $$
	  
	  $$
		min_{lineLength} = 40 
		\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;  
		max_{lineGap} = 20
	$$
	
6. **Average Lines** : With the lines identified in the region of interest the final step is to obtain only 2 lines that will delimit the lane. As seen in the trapezoid figure the line on the left and on the right will have different slopes, so two arrays were used to save each of the lines:
	$$
	line \in 
	\begin{cases}
	\overline{lines_n}, & if & -\infty < m < 0, & |m| > \alpha\\
	\overline{lines_p}, & if & 0 < m < \infty, & |m| > \alpha
	\end{cases}
	$$
Where $\alpha$ represents a threshold to ignore non-related lines. Each array stores the lines coefficients ($m, b$), then a single $m$ and $b$ coefficient is calculated in each array by averaging.

$$
\overline m = \dfrac{\sum^i_{n} m_i}{n} 
\;\;\;\;\;
\overline b = \dfrac{\sum^i_{n} m_i}{n} 
$$

In order to build a line in the image, we need the coordinates of two points, for this the line equation was arranged in the following way. 
$$x = \dfrac{y - b}{m}$$

The only remaining pieces to complete the puzzle are the $y_1$ and $y_2$ values; $y_1$ was taken as the minimum $y$ point between all the lines detected in the Hough Transform and $y_2$ refers to the image height.

$$
y_1 = \min\limits_i
(line^i_{y_1}, line^i_{y_2})
\;\;\;\;\;\;\;
y_2 = img_h
$$


### 2. Shortcomings

There are several flaws in this approach:

* It is not illumination invariant, so the lack of contrast and light would result in a malfunction of the program.

* A fixed camera position and consistent image size are assumed, so positioning the camera at a different angle and handling different image sizes would have undesirable results.

* We are only detecting straight lines, therefore in pronounced and prolonged curves the algorithm could present anomalies.


### 3. Improvements

Each point of the pipeline can be improved.

1. **Grayscale** : There are multiple ways to convert from RGB to grayscale, finding the right one for each situation, would help to better line detection. Similarly, a histogram equalization or contrast corrector before colorspace conversion could improve the results. Finally, the use of a different color space such as HSV (illumination invariance) would make the model more robust.

2. **Gaussian blur** : The sigma and kernel size values play an important role for noise reduction, being able to pick the right values in a smart way would lead to better results.

3. **Canny edge detection** : Choosing the right threshold values have a critical role for the good performance of the application, the Otsu thresholding method could help to find the best values for each case.

4. **Mask region of interest** : The way trapezoid vertices were calculated works well but only for this size and perspective of the camera, a smart method that allows to find the points of interest (for example extrapolating the longer straight lines to find the coordinates in x) would make the model more robust to interact with multiple image sizes and camera angles.

5. **Hough Transform** : The Hough transform presents a great challenge to find the best parameters, methods such as *Grid Search* can be useful to find the best possible combination within a limited number of options, however this type of approach would never work in a real-time scenario where the values must be calculated at every frame.

6. **Average lines** :  Although functional, averaging all the lines found is still very simple, the model can be improved by suppressing outliers or extrapolating a line, which we are sure belongs to the lane.