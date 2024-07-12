SVM:
% Load the leaf images from the Healthy and Diseased folders healthyFolder = 'Healthy'; % Path to Healthy folder diseasedFolder = 'Diseased'; % Path to Diseased folder

healthyImages = imageDatastore(healthyFolder); diseasedImages = imageDatastore(diseasedFolder);

% Create labels for the images (0 for healthy, 1 for diseased) healthyLabels = zeros(numel(healthyImages.Files), 1); diseasedLabels = ones(numel(diseasedImages.Files), 1);

% Combine the data and labels
allImages = [healthyImages.Files; diseasedImages.Files]; allLabels = [healthyLabels; diseasedLabels];

% Initialize an array to store color histograms
numBins = 256; % Number of bins for each color channel
colorHistograms = zeros(numel(allImages), 3 * numBins); % 3 channels (RGB)

% Compute color histograms for each image for i = 1:numel(allImages)
img = imread(allImages{i});
% Split the image into RGB channels redChannel = img(:, :, 1); greenChannel = img(:, :, 2); blueChannel = img(:, :, 3);
% Compute histograms for each channel redHist = imhist(redChannel, numBins); greenHist = imhist(greenChannel, numBins); blueHist = imhist(blueChannel, numBins);
% Concatenate histograms
colorHistograms(i, :) = [redHist; greenHist; blueHist]';

end
% Load the mixed test images
mixedFolder = 'Mixed'; % Path to Mixed folder mixedImages = imageDatastore(mixedFolder);

% Extract features (color histograms) from the mixed test images mixedFeatures = zeros(numel(mixedImages.Files), 3 * numBins); for i = 1:numel(mixedImages.Files)
img = imread(mixedImages.Files{i}); redChannel = img(:, :, 1); greenChannel = img(:, :, 2); blueChannel = img(:, :, 3);
redHist = imhist(redChannel, numBins); greenHist = imhist(greenChannel, numBins); blueHist = imhist(blueChannel, numBins);
mixedFeatures(i, :) = [redHist; greenHist; blueHist]'; end

% Train the SVM model
svmModel = fitcsvm(colorHistograms, allLabels, 'KernelFunction', 'linear');

% Predict the labels for the mixed test images mixedPredictions = predict(svmModel, mixedFeatures);

% Calculate the percentage of diseased leaves in the Mixed folder numDiseased = sum(mixedPredictions);
totalLeaves = numel(mixedImages.Files); percentageDiseased = (numDiseased / totalLeaves) * 100;
fprintf('Percentage of diseased leaves in the Mixed folder: %.2f%%\n', percentageDiseased);
