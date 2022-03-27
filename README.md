<!--21700150 김인웅-->
Lab1
======
목표 : 스마트 팩토리를 위해 이미지 안의 물체 구별하기  

주어진 이미지

![Lab_GrayScale_TestImage](https://user-images.githubusercontent.com/80805040/160271677-0a9c1d2c-adf8-4d87-aa70-cbe724c4cf49.jpg) 

Bolt M5, Bolt M6, Square Nut M5, Hexa Nut M5, Hexa Nut M6를 분류해야 한다.  

##### Filter  
좌측 아래의 두개의 Square Nut M5가 붙어 있는 형상을 분류하기 위해 Laplacian 필터를 사용하였다. 이 때 선택한 kernal size 는 5이다.
Laplacian 필터가 적용된 이미지  

![Laplacian_Filter](https://user-images.githubusercontent.com/80805040/160271690-b247cede-f9a1-444a-b170-ef41e8ce5064.png)

Laplacian 필터의 사용으로 인해 많은 노이즈가 생긴 것이 보인다. 따라서 Median 필터를 사용하여 노이즈를 줄여준다. 이 때 선택한 kernel size 는 21이다.
Median 필터가 적용된 이미지

![Median_Filter](https://user-images.githubusercontent.com/80805040/160271691-07590d7e-dd19-4d93-9168-9fb388d6cc9a.png)  

##### Threshold  
Contour를 위해 Threshold를 조정하였다. 이때 왼쪽 아래의 Hexa Nut M5가 잘 나타날 수 있도록 선택한 값은 140이다.
Threshold를 적용한 뒤의 이미지

![Threshold](https://user-images.githubusercontent.com/80805040/160271697-9b437ac4-c8ce-4775-8922-c66f9400ed13.png)   

##### Morphology  
Threshold 이후 이미지를 살펴보면 이미지가 끊어 있는 것을 확인 할 수 있다. 이를 해결하기 위해 Morphology로 Close를 한 번 수행하고, 노이즈를 줄이기 위해 Erode를 수행한 후 최종적으로 Dilate를 수행하여 완전한 이미지를 얻었다. Close 에 선택한 kernel size는 31, Erode에 선택한 kernel size는 7, 그리고 Dilate에 선택한 kernel size는 5이다. 
Morphology 이후의 이미지
![Morphology](https://user-images.githubusercontent.com/80805040/160271700-c1e65778-9cc1-401b-82b2-ab61f9bde8be.png)  

##### Contours  
최종적으로 얻어낸 이미지를 Contours를 이용해 부품 개수를 얻어낸다.
최종적으로 Contours를 적용한 이미지

![Contours](https://user-images.githubusercontent.com/80805040/160271707-67a70796-4829-4dc1-8755-843bc3f69394.png)

이때 contourArea와 arcLength를 이용하여 찾아낸 각 물체들의 면적과 길이를 얻을 수 있다. 각 부품들의 크기와 킬이에는 확실한 차이가 있음으로 그것을 이용하여 부품을 구별 해 내었다.
지정한 범위는 아래와 같다.
Hexa Nut M5 : 2000 < Area < 3500, 150 < Length < 225.93
Square Nut M5 : 2400 < Area < 4400, 225.93 < Length < 270
Hexa Nut M6 : 5000 < Area < 5700, 270 < Length < 300
Bolt M5 : 3600 < Area < 5200, 420 < Length < 450
Bolt M6 : 6700 < Area < 7700, 550 < Length < 600
최종 결과

![Conclusion](https://user-images.githubusercontent.com/80805040/160271710-685b785a-ec0d-48a2-a6e0-466b83a84024.png)  

### Appendix
##### 코드
    #include <opencv2/opencv.hpp>
    #include <iostream>
    
    using namespace std;
    using namespace cv;
    
    int element_shape = MORPH_RECT;	
    int n = 7;
    int m = 5;
    int j = 31;
    Mat element1 = getStructuringElement(element_shape, Size(n, n));
    Mat element2 = getStructuringElement(element_shape, Size(m, m));
    Mat element3 = getStructuringElement(element_shape, Size(j, j));
    Mat src, dst, t_dst, dst_morph;
    
    void main()
    {
        src = cv::imread("Lab_GrayScale_TestImage.jpg", 0);
    
        int kernel_size = 5;
        int scale = 1;
        int delta = 0;
        int ddepth = CV_16S;
    
        cv::Laplacian(src, dst, ddepth, kernel_size, scale, delta, cv::BORDER_DEFAULT);
        src.convertTo(src, CV_16S);
        cv::Mat result_laplacian = src - dst;
        result_laplacian.convertTo(result_laplacian, CV_8U);
        /*namedWindow("Laplacian", CV_WINDOW_NORMAL);
        cv::imshow("Laplacian", result_laplacian);*/
    
        cv::medianBlur(result_laplacian, dst, 21);
        /*namedWindow("Median", CV_WINDOW_NORMAL);
        imshow("Median", dst);*/
    
        cv::threshold(dst, t_dst, 140, 255, 0);
        /*namedWindow("tsh", WINDOW_NORMAL);
        imshow("tsh", t_dst);*/
    
        cv::morphologyEx(t_dst, dst_morph, CV_MOP_CLOSE, element3);
        cv::erode(dst_morph, dst_morph, element1);
        cv::dilate(dst_morph, dst_morph, element2);
        /*namedWindow("morphed", WINDOW_NORMAL);
        imshow("morphed", dst_morph);*/
    
        vector<vector<Point>> contours;
    
        /// Find contours
        findContours(dst_morph, contours, CV_RETR_EXTERNAL, CV_CHAIN_APPROX_NONE, Point(0, 0));
    
        /// Draw all contours excluding holes
        Mat drawing(dst_morph.size(), CV_8U, Scalar(255));
        drawContours(drawing, contours, -1, Scalar(0), CV_FILLED);
    
        int HM5 = 0;
        int HM6 = 0;
        int SM5 = 0;
        int M5 = 0;
        int M6 = 0;
    
        for (int i = 0; i < contours.size(); i++)
        {
            double Area = contourArea(contours[i]);
            double Length = arcLength(contours[i], true);
    
            if (2000.00 < Area && Area < 3500.00 && 150.00 < Length && Length < 225.93) {
                HM5 += 1;
            }
    
            else if (2400.00 < Area && Area < 4400.00 && 225.93 < Length && Length < 270.00) {
                SM5 += 1;
            }
    
            else if (5000.00 < Area && Area < 5700.00 && 270.00 < Length && Length < 300.00) {
                HM6 += 1;
            }
    
            else if (3600.00 < Area && Area < 5200.00 && 420.00 < Length && Length < 450.00) {
                M5 += 1;
            }
    
            else if (6700.00 < Area && Area < 7700.00 && 550.00 < Length && Length < 600.00) {
                M6 += 1;
            }
    
            else {
                continue;
            }
        }
    
        int sum = HM5 + HM6 + SM5 + M5 + M6;
    
        cout << " Number of Hexa Nut M5 are =" << HM5 << endl;
        cout << " Number of Hexa Nut M6 are =" << HM6 << endl;
        cout << " Number of Square Nut M5 are =" << SM5 << endl;
        cout << " Number of Bolt M5 are =" << M5 << endl;
        cout << " Number of Bolt M6 are =" << M6 << endl;
        cout << " Number of All Elements are =" << sum << endl;
    
        namedWindow("contour", WINDOW_NORMAL);
        imshow("contour", drawing);
    
        while (true) {
            int c = waitKey(20);
            if (c == 27)
                break;
        }
    }


##### Flow Chart

![Flow_Chart](https://user-images.githubusercontent.com/80805040/160271711-7211a88a-cb9a-4479-ace9-0da2787c5529.png)
