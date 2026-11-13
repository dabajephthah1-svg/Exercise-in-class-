# Exercise-in-class-

%% MATLAB IMAGE PROCESSING - IN-CLASS EXERCISES
% Exercises 1-6
% Save as: exercises.mlx

clc;
clear;
close all;

fprintf('========================================\n');
fprintf('MATLAB IMAGE PROCESSING EXERCISES\n');
fprintf('========================================\n');

%% EXERCISE 1: PIXELS AND QUANTIZATION

fprintf('\nEXERCISE 1: PIXELS AND QUANTIZATION\n');

I1 = imread('cameraman.tif');

fprintf('Image size: %d x %d\n',size(I1,1),size(I1,2));
fprintf('Image class: %s\n',class(I1));

I = im2double(I1);

% Quantization to 4, 2 and 1 bit
bits = [4 2 1];

figure('Name','Exercise 1 - Quantization');

for k = 1:length(bits)
    L = 2^bits(k);
    Q = floor(I*L)/(L-1);
    Q = min(Q,1);

    subplot(1,3,k);
    imshow(Q);
    title(sprintf('%d-bit image',bits(k)));
end

% Downsampling by 8
small1 = I(1:8:end,1:8:end);
small2 = imresize(I,1/8);

figure('Name','Exercise 1 - Downsampling');

subplot(1,2,1);
imshow(small1);
title('Without imresize');

subplot(1,2,2);
imshow(small2);
title('With imresize');

fprintf('\nThink about:\n');
fprintf(['At 1 bit, considerable detail is lost, although ' ...
    'the main shapes remain recognizable.\n']);
fprintf(['Direct downsampling creates aliasing because high-frequency ' ...
    'details are not properly filtered.\n']);


%% EXERCISE 2: DEGRADE AN IMAGE

fprintf('\nEXERCISE 2: DEGRADE AN IMAGE\n');

I = im2double(imread('cameraman.tif'));

% Gaussian noise
gaussianNoise = imnoise(I,'gaussian',0,0.01);

% Salt and pepper noise
spNoise = imnoise(I,'salt & pepper',0.05);

% Motion blur
motionKernel = fspecial('motion',15,30);
motionBlur = imfilter(I,motionKernel,'conv','replicate');

% Brightness ramp
[rows,cols] = size(I);

ramp = repmat(linspace(0.3,1.5,cols),rows,1);

brightnessImage = I.*ramp;
brightnessImage = min(brightnessImage,1);

% Display degraded images
figure('Name','Exercise 2 - Image Degradation');

subplot(2,3,1);
imshow(I);
title('Original');

subplot(2,3,2);
imshow(gaussianNoise);
title('Gaussian Noise');

subplot(2,3,3);
imshow(spNoise);
title('Salt and Pepper Noise');

subplot(2,3,4);
imshow(motionBlur);
title('Motion Blur');

subplot(2,3,5);
imshow(brightnessImage);
title('Brightness Ramp');

fprintf('\nThink about:\n');
fprintf(['Gaussian and salt-and-pepper noise are additive noise ' ...
    'models. The brightness ramp is multiplicative.\n']);
fprintf(['The median filter is particularly effective at removing ' ...
    'salt-and-pepper noise.\n']);


%% EXERCISE 3: CONTRAST AND GEOMETRY

fprintf('\nEXERCISE 3: CONTRAST AND GEOMETRY\n');

P = imread('pout.tif');

% Original histogram
figure('Name','Exercise 3 - Histogram');

imhist(P);
title('Original Pout Histogram');
xlabel('Intensity');
ylabel('Frequency');

% Contrast adjustment
adjusted = imadjust(P);

% Gamma correction
gamma05 = imadjust(P,[],[],0.5);
gamma2 = imadjust(P,[],[],2);

% Histogram equalization
equalized = histeq(P);

% Display contrast results
figure('Name','Exercise 3 - Contrast');

subplot(2,3,1);
imshow(P);
title('Original');

subplot(2,3,2);
imshow(adjusted);
title('Imadjust');

subplot(2,3,3);
imshow(gamma05);
title('Gamma 0.5');

subplot(2,3,4);
imshow(gamma2);
title('Gamma 2');

subplot(2,3,5);
imshow(equalized);
title('Histogram Equalization');

% Extract 40 x 40 patch
patch = P(100:139,100:139);

% Enlarge patch
nearest = imresize(patch,5,'nearest');
bicubic = imresize(patch,5,'bicubic');

figure('Name','Exercise 3 - Interpolation');

subplot(1,2,1);
imshow(nearest);
title('Nearest Interpolation');

subplot(1,2,2);
imshow(bicubic);
title('Bicubic Interpolation');

fprintf('\nThink about:\n');
fprintf(['Gamma 0.5 brightens the image because intensity values ' ...
    'between 0 and 1 increase when raised to a power below 1.\n']);
fprintf(['Bicubic interpolation uses neighbouring pixels to calculate ' ...
    'new pixels, producing smoother results than nearest neighbour.\n']);


%% EXERCISE 4: FILTER AND COMPARE

fprintf('\nEXERCISE 4: FILTER AND COMPARE\n');

I = im2double(imread('cameraman.tif'));

% Create noisy images
noisyImages = {
    imnoise(I,'gaussian',0,0.01), ...
    imnoise(I,'salt & pepper',0.05)
};

noiseNames = {'Gaussian','Salt and Pepper'};

% Define filters
meanKernel = fspecial('average',[3 3]);
gaussKernel = fspecial('gaussian',[5 5],1);

filterNames = {'Mean','Gaussian','Median'};

PSNR = zeros(6,1);
results = cell(6,1);

row = 1;

for n = 1:2

    noisy = noisyImages{n};

    meanResult = imfilter(noisy,meanKernel,'replicate');

    gaussianResult = imfilter(noisy,gaussKernel,'replicate');

    medianResult = medfilt2(noisy,[3 3]);

    filtered = {
        meanResult,...
        gaussianResult,...
        medianResult
    };

    figure('Name',['Exercise 4 - ' noiseNames{n}]);

    for f = 1:3

        subplot(1,3,f);
        imshow(filtered{f});

        title([noiseNames{n} ' - ' filterNames{f}]);

        % Calculate PSNR
        mse = mean((I(:)-filtered{f}(:)).^2);

        PSNR(row) = 10*log10(1/mse);

        results{row} = [noiseNames{n} ' - ' filterNames{f}];

        row = row+1;

    end
end

% Create PSNR table
T = table(results,PSNR,...
    'VariableNames',{'Noise_and_Filter','PSNR_dB'});

fprintf('\nPSNR COMPARISON TABLE:\n');
disp(T);

% Original and blurred image spectrum
motionKernel = fspecial('motion',15,30);

blurred = imfilter(I,motionKernel,'conv','replicate');

F1 = fftshift(fft2(I));
F2 = fftshift(fft2(blurred));

figure('Name','Exercise 4 - Spectrum');

subplot(1,2,1);
imshow(log(1+abs(F1)),[]);
title('Original Spectrum');

subplot(1,2,2);
imshow(log(1+abs(F2)),[]);
title('Motion Blurred Spectrum');

fprintf('\nThink about:\n');
fprintf(['Median filtering generally works best for salt-and-pepper ' ...
    'noise. Mean and Gaussian filters can reduce Gaussian noise.\n']);
fprintf(['Blurring reduces high-frequency components, making fine ' ...
    'details and sharp edges weaker in the spectrum.\n']);


%% EXERCISE 5: EDGE DETECTION

fprintf('\nEXERCISE 5: EDGE DETECTION\n');

I = imread('cameraman.tif');

% Sobel edge detection
sobelEdges = edge(I,'sobel');

% Canny with different sigma values
canny1 = edge(I,'canny',[],1);
canny2 = edge(I,'canny',[],2);
canny3 = edge(I,'canny',[],3);

figure('Name','Exercise 5 - Edge Detection');

subplot(2,2,1);
imshow(sobelEdges);
title('Sobel');

subplot(2,2,2);
imshow(canny1);
title('Canny Sigma 1');

subplot(2,2,3);
imshow(canny2);
title('Canny Sigma 2');

subplot(2,2,4);
imshow(canny3);
title('Canny Sigma 3');

% Add Gaussian noise
I = im2double(I);

noisy = imnoise(I,'gaussian',0,0.01);

% Canny without smoothing
edgesWithout = edge(noisy,'canny');

% Gaussian smoothing followed by Canny
smoothed = imgaussfilt(noisy,1);

edgesWith = edge(smoothed,'canny');

% Count edge pixels
countWithout = nnz(edgesWithout);
countWith = nnz(edgesWith);

fprintf('\nEDGE PIXEL COUNTS:\n');

fprintf('Without smoothing: %d\n',countWithout);
fprintf('With smoothing: %d\n',countWith);

fprintf('Difference: %d\n',countWithout-countWith);

if countWithout > 0
    reduction = 100*(countWithout-countWith)/countWithout;
    fprintf('Percentage reduction: %.2f%%\n',reduction);
end

% Display comparison
figure('Name','Exercise 5 - Smoothing');

subplot(1,2,1);
imshow(edgesWithout);
title('Without Smoothing');

subplot(1,2,2);
imshow(edgesWith);
title('With Gaussian Smoothing');

fprintf('\nThink about:\n');
fprintf(['Increasing sigma produces stronger smoothing and can remove ' ...
    'fine details and small edges.\n']);
fprintf(['Smoothing normally reduces noise-related false edges, ' ...
    'although some real edges can also disappear.\n']);


%% EXERCISE 6: SEGMENT AND MEASURE

fprintf('\nEXERCISE 6: SEGMENT AND MEASURE\n');

I = imread('coins.png');

% Otsu thresholding
level = graythresh(I);

BW = imbinarize(I,level);

fprintf('Otsu threshold: %.4f\n',level);

% Morphological operations
se = strel('disk',2);

opened = imopen(BW,se);

filled = imfill(opened,'holes');

cleaned = bwareaopen(filled,50);

% Label connected components
[L,numCoins] = bwlabel(cleaned);

% Measure areas
measurements = regionprops(L,'Area');

areas = [measurements.Area]';

fprintf('\nNumber of detected coins: %d\n',numCoins);

% Print area of each coin
for k = 1:numCoins

    fprintf('Coin %d area: %.0f pixels\n',k,areas(k));

end

% Find largest coin
if ~isempty(areas)

    [largestArea,largestIndex] = max(areas);

    fprintf('\nLargest coin: %d\n',largestIndex);

    fprintf('Largest coin area: %.0f pixels\n',largestArea);

else

    fprintf('No coins were detected.\n');

end

% Display segmentation results
figure('Name','Exercise 6 - Segmentation');

subplot(2,2,1);
imshow(I);
title('Original Image');

subplot(2,2,2);
imshow(BW);
title('Otsu Threshold');

subplot(2,2,3);
imshow(filled);
title('After Filling');

subplot(2,2,4);
imshow(label2rgb(L));
title('Labelled Coins');

fprintf('\nThink about:\n');
fprintf(['Opening removes small foreground objects and thin connections. ' ...
    'Filling removes holes inside objects.\n']);
fprintf(['Area opening removes small connected objects. Touching coins ' ...
    'may merge into one connected component.\n']);


%% FINAL SUMMARY

fprintf('\n========================================\n');
fprintf('ALL SIX EXERCISES COMPLETED\n');
fprintf('========================================\n');

fprintf('Exercise 1: Quantization and downsampling completed.\n');
fprintf('Exercise 2: Image degradation completed.\n');
fprintf('Exercise 3: Contrast and geometry completed.\n');
fprintf('Exercise 4: Filtering and PSNR completed.\n');
fprintf('Exercise 5: Edge detection completed.\n');
fprintf('Exercise 6: Segmentation and measurement completed.\n');

fprintf('\nRemember to save and export this Live Script as PDF.\n');
