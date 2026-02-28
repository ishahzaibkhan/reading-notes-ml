This video from CampusX, presented by Nitish, provides a comprehensive explanation of Transformer Inference, focusing on how a trained Transformer model makes predictions (0:00). The video continues the "100 Days of Deep Learning" playlist, marking the final discussion on the Transformer architecture.

Here are the detailed notes from the video:

1. Introduction and Overview of Transformer Architecture (0:00 - 2:09)

The video is the last in a series dedicated to the Transformer architecture within the "100 Days of Deep Learning" playlist (0:04).
Previously, the series covered the encoder and decoder components in detail, explaining their internal workings and overall architecture (0:31-1:17).
The primary focus of this video is to discuss how the Transformer architecture behaves during the inference (prediction) stage, which hasn't been covered in previous videos (1:26-2:09).
2. Setup for Transformer Inference (2:10 - 3:36)

The speaker sets up a machine translation task from English to Hindi sentences (2:23).
The dataset consists of input English sentences and corresponding Hindi output sentences (2:34).
A Transformer architecture has been trained on this data, and its weights and biases are ready for predictions (2:50).
The specific query sentence used as an example is "We are friends" to be translated into Hindi (3:10). The video will then step-by-step explain the inference process for this query (3:21).
3. Key Differences: Encoder vs. Decoder During Inference (3:37 - 6:21)

Encoder Behavior: The encoder behaves identically during both training and inference stages (4:23). Therefore, no in-depth discussion about the encoder is needed for inference (4:40-6:03).
Decoder Behavior: This is where the main difference lies (4:47).
During training, the decoder behaves in a non-autoregressive manner (5:15), processing all words of the output sentence simultaneously since they are already known (5:01-5:22).
During inference, the decoder behaves in an autoregressive manner (5:42), predicting one word at a time over multiple time steps because the output sentence is unknown (5:28-5:46).
The video's going-forward focus will be on the decoder's working during inference (6:14).
4. Encoder Processing (6:22 - 8:46)

The query sentence "We are friends" is first sent to the encoder (6:40).
The encoder's process is consistent with training (6:50), involving:
Tokenization: Breaking the sentence into tokens (e.g., "We," "are," "friends") (7:14).
Embedding Calculation: Converting tokens into numerical embeddings (7:24).
Positional Encoding: Adding positional information to embeddings (7:30).
Multi-head Attention: Extracting context (7:35).
Feed-forward Network: Applying non-linearities (7:38).
Add & Normalize Layers: Applied between steps (7:43).
This entire process is repeated six times for the six encoder blocks (7:49).
The final output from the sixth encoder block provides a context vector for each token (e.g., for "We," "are," "friends") (7:54-8:20). These context vectors capture the contextual understanding of each word (8:23-8:36).
At this stage, the Transformer has understood the English sentence (8:39).
5. Decoder's First Word Prediction (8:47 - 24:46)

The decoder's role is to print the Hindi sentence word by word (8:49).
The decoder processes output step by step, with each step yielding one word (9:49).
Initialization: A special token, "SOS" (Start of Sentence), is sent to the decoder to initiate the process (10:17).
SOS Token Processing within the Decoder Block:
Embedding Layer: The "SOS" token is vectorized into a 512-dimension embedding (10:51).
Positional Encoding Layer: Positional encoding is added, even for a single token (11:12). This results in the input vector x1 (11:31).
Masked Self-Attention Block:
x1 is sent to this block (12:06).
Query (Q), Key (K), and Value (V) vectors are calculated by dot product with Wq, Wk, Wv matrices (12:30).
Dot product of Q and K is performed, scaled, and then Softmax is applied to get attention scores (13:06). This score indicates the similarity of "SOS" with itself (13:22).
The attention score is multiplied by the Value vector to get z1, a 512-dimension vector (13:35).
Add & Normalize Layer: z1 is added back to x1 (residual connection) to get z1' (14:10), which is then layer-normalized to z1_norm (14:34).
Cross-Attention Block:
This crucial block calculates attention scores between two sequences: the decoder's current sequence (currently "SOS") and the encoder's contextual vectors ("We are friends") (15:00-15:28).
Query vectors are extracted from the decoder's sequence (z1_norm) (16:26).
Key and Value vectors are extracted from the encoder's output (16:30-17:42).
Dot products between queries and keys (Q_SOS with K_V, K_R, K_Friends) yield scalar attention scores (17:46-18:06).
These scalars are scaled, Softmax is applied to get weights (e.g., w11, w12, w13), which are then multiplied by the respective Value vectors (e.g., V_V, V_R, V_Friends) (18:09-18:31).
This results in zC1 (Cross Attention output vector), which represents the contextual understanding of "SOS" concerning the English sentence (18:34-18:43).
Add & Normalize Layer (after Cross-Attention): zC1 is added to z1_norm (residual connection) to get zC1' (19:00), which is then layer-normalized to zC1_norm (19:07).
Feed-Forward Network:
zC1_norm is sent to a feed-forward neural network (19:28).
This network has a hidden layer with 2048 neurons (ReLU activation) and an output layer with 512 neurons (linear activation) (20:07-20:32).
This performs a non-linear transformation, yielding y1 (21:19-21:26).
Add & Normalize Layer (after Feed-Forward): y1 is added to zC1_norm (residual connection) to get y1' (21:35), which is then layer-normalized to y1_norm (21:45).
Repetition for Six Decoder Blocks: This entire process (embedding to output) is repeated six times through the six decoder blocks (22:08-22:38). The final output from the sixth decoder is yF1_norm (22:38-22:45).
Output Layer (Linear + Softmax):
yF1_norm (512-dimension vector) is sent to a linear layer (22:56).
This layer's output dimension (V) corresponds to the size of the Hindi vocabulary (unique words in the dataset) (23:10-23:39). Each neuron in this layer represents a unique Hindi word (23:36-23:42).
The raw output (logits) are then passed through a Softmax function to get a probability distribution over the entire Hindi vocabulary (23:49-24:17).
The word with the highest probability is selected as the first output word (24:20-24:29). In the example, this word is "हम" (hum - "we") (24:30-24:46).
6. Decoder's Subsequent Word Predictions (24:47 - 43:29)

Autoregressive Nature: The decoder is autoregressive, meaning it uses previously generated tokens as input for the next prediction (24:51-24:59).
Second Time Step:
Input: The input to the decoder for the second time step will be "SOS" + "हम" (the previous output) (25:11-25:34).
The entire process described in section 5 (embedding, positional encoding, masked self-attention, cross-attention, feed-forward, add & normalize, repetition through 6 blocks) is repeated with these two tokens (25:38-38:45).
Masked Self-Attention with Multiple Tokens:
For "SOS" and "हम", query, key, and value vectors are calculated (27:18-27:39).
Attention matrix is calculated (SOS-SOS, SOS-हम, हम-SOS, हम-हम) (28:14-28:36).
Crucially, masking is applied even during inference (28:48). This prevents tokens from attending to future tokens (e.g., "SOS" cannot see "हम") (28:56-29:55). This is vital for consistency with training and maintaining prediction quality (30:29-31:07).
Outputs z1 and z2 are generated (31:55-32:01).
Cross-Attention with Multiple Tokens: The two decoder vectors (z1_norm, z2_norm) interact with the encoder's three contextual vectors to calculate attention scores (33:01-37:05). This results in zC1 and zC2.
Feed-Forward Network: Processes zC1_norm and zC2_norm to produce y1 and y2 (37:06-37:52).
Final Output: After passing through all six decoder blocks, two final vectors, yF1_norm and yF2_norm, are produced (38:36-38:48).
Prediction: At this point, only the last vector (yF2_norm, corresponding to "हम") is sent to the linear + Softmax layer (39:03-40:00). The yF1_norm (corresponding to "SOS") is ignored because its output was already produced in the previous step (40:00).
The word with the highest probability is then chosen. In the example, this is "दोस्त" (dost - "friend") (40:27-40:32).
Third and Subsequent Time Steps:
The input for the third step would be "SOS" + "हम" + "दोस्त" (40:57-41:07).
The entire process repeats (embedding, masked self-attention with all previous tokens, cross-attention, etc.), generating three output vectors from the final decoder block (41:07-41:52).
Again, only the last vector (corresponding to "दोस्त") is passed to the linear + Softmax layer (41:57-42:09).
This continues until the model predicts an "EOS" (End of Sentence) token (42:15-43:08).
Upon predicting "EOS", the inference process stops (43:08-43:13).
The final translated sentence for "We are friends" is "हम दोस्त है" (hum dost hai - "we are friends") (43:17-43:29).
7. Conclusion and Key Takeaways (43:30 - 45:12)

The speaker reiterates that the encoder works identically during training and inference (43:36-43:39).
The main difference is the autoregressive behavior of the decoder during inference, processing one token at a time (43:40-43:58).
Two crucial points to remember:
Tokens are incrementally added as input in subsequent time steps (e.g., 1 token, then 2, then 3, etc.) (44:02-44:14).
Masking is always applied during inference, mirroring the training process, to maintain data consistency and prediction quality (44:14-44:30).
The video concludes the Transformer architecture discussion, and future videos will cover other architectures like GPT and BERT, as well as Transformer fine-tuning (44:35-44:59).
