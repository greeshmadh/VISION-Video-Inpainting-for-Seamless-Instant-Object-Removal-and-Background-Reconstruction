# VISION-Video-Inpainting-for-Seamless-Instant-Object-Removal-and-Background-Reconstruction

This repository provides a solution for masking out specific objects in a video based on a text prompt and then inpainting the masked regions. It leverages Detectron2 for object detection and segmentation, CLIP for image-text similarity, and a random walks algorithm for inpainting. 

##PUBLICATION LINK:- [IEEE](https://ieeexplore.ieee.org/document/11081337 "VISION-Video-Inpainting-for-Seamless-Instant-Object-Removal-and-Background-Reconstruction")


## Setup

1.  **Mount Google Drive:** The code assumes your video file is stored in Google Drive. Ensure you have mounted your Google Drive to `/content/drive`. The first code cell in the notebook handles this.
2.  **Install Dependencies:** The second code cell installs all the necessary libraries, including `torch`, `torchvision`, `opencv-python`, `CLIP`, `pyyaml`, `detectron2`, and `spacy`.
3.  **Download Spacy Model:** The second code cell also downloads the `en_core_web_sm` spacy model.

## How to Use

1.  **Upload your video:** Place your video file in your Google Drive and update the `video_path` variable in the consolidated code cell to point to your video file.
2.  **Run the consolidated code cell:** Execute the code cell that contains the entire video processing pipeline.
3.  **Enter text prompt:** When prompted, enter a text prompt describing the object you want to mask out (e.g., 'red car', 'person on the left'). The code will parse this prompt to identify the object type, color, and position (if specified).

The code will then:
*   Extract frames from your video.
*   Segment the specified object in each frame and create masks.
*   Inpaint the masked regions of the first frame using a random walks (graph) algorithm.
*   Fill the masked pixels in the result frames using the inpainted first frame. This replaces the areas in each frame that were masked out (turned black) during the segmentation step with the content from the inpainted first frame, effectively filling in the removed object using bitwise operations.
*   Combine the processed frames into a new video with the specified object masked out and the background inpainted.

## Output

The processed video with the masked object and inpainted background will be saved in the same directory as your input video file, with a filename like `[your_video_filename]_output.mp4`.

Intermediate directories will also be created in the same location to store extracted frames, masks, and intermediate results.

## Customization

*   **`video_path`**: Change this variable to point to your video file.
*   **`fps`**: Adjust the `fps` parameter in the `extract_frames` function to change the frame rate of the extracted frames.
*   **`cfg.MODEL.ROI_HEADS.SCORE_THRESH_TEST`**: Modify this threshold in the consolidated code cell to adjust the confidence level for object detection.
*   **`similarity_threshold`**: Change this threshold in the `segment_frames` function to adjust the similarity score required for an object to be considered a match with the text prompt.
*   **`low_threshold` and `high_threshold`**: Adjust these thresholds in the `create_mask_for_inpainting` function to control which pixels are considered damaged and require inpainting.
*   **`frame_rate`**: Adjust the `frame_rate` parameter in the `frames_to_video` function to change the frame rate of the output video.


### Random Walk Inpainting

Random Walk inpainting is used to fill missing regions (holes) in an image by propagating surrounding pixel values smoothly into the masked areas.

1. **Purpose** – Removes masked areas (e.g., objects) from an image by generating a smooth background that blends naturally with known pixels.

2. **Concept** – Treats the image as a graph where each pixel is connected to its neighbors. A "random walker" starting at an unknown pixel moves randomly until it reaches a known pixel; the pixel's value is determined by the probabilities of reaching each known pixel first.

3. **Mathematics** – Solves Laplace’s equation over unknown pixels with boundary values from known pixels, creating a smooth interpolation from the edges inward.

4. **Steps**:
   - Identify known and unknown pixels from the mask.
   - Build a Laplacian matrix representing pixel connectivity.
   - Solve a linear system to estimate values for unknown pixels.
   - Replace missing pixels with these computed values.

5. **Advantages** – Produces smooth, seamless fills for small to medium missing regions and is effective for background reconstruction.

6. **Usage in Project** – The first masked frame is inpainted using Random Walk to create a clean background (`final_<video_name>/`) before processing subsequent frames.
