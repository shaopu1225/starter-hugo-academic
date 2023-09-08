---
title:		An intro survey of Federated Learning Privacy Protection
subtitle:	An intro survey of Federated Learning Privacy Protection
summary:	notes of the Future of Decentralization, AI, and Computing Summit at UC Berkeley
date:		2023-08-27
lastmod:	2023-09-07
author:		shaopu
draft: 		false

tags:
    - Artificial Intelligence
    - Federated Learning

categories:
    - Talks
---

> **Federated learning** (also known as **collaborative learning**) is a machine learning technique that trains an algorithm via multiple independent sessions, each using its own dataset. This approach stands in contrast to traditional centralized machine learning techniques where local datasets are merged into one training session, as well as to approaches that assume that local data samples are identically distributed.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-09-08-025411.png" alt="Federated Learning: Collaborative Machine Learning without Centralized  Training Data – Google Research Blog" style="zoom:80%;" />

The procedure of Federated Learning can be descripted using the following figure:

<img src="https://img2022.cnblogs.com/blog/2842354/202204/2842354-20220420121840485-1765157162.png" alt="img" style="zoom:80%;" />

Federated learning can be divided into horizontal federated learning, vertical federated learning and federated transfer learning according to the distribution of data.

- **Horizontal Federated Learning**: Suitable when the user features of the two datasets overlap a lot, but the users overlap little. This pattern allows the increasing of user sample size and improvement of model accracy.
- **Vertical Federated Learning**: Available when the user features of the two datasets overlap little, but the users overlap a lot. This pattern usually can be leveraged to increase the user feature dimension.
- **Federated transfer Learning**: Users and user features of the two datasets both rarely overlap. Transfer learning must be introduced to solve the problems of small unilateral data size and small label samples.

![image-20230907195323234](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-09-08-025323.png)

As described above, Federated learning is designed to empower machine learning models to improve and adapt using data from various decentralized devices or data sources, all while ensuring that the data remains on those devices and is kept private. This approach eliminates the necessity to gather sensitive data into a central location, which could otherwise jeopardize data privacy and security.

Based on this initiative, privacy becomes a very important topic in the Federated Learning. Although the local data ensure some extents of "privacy" in the training process, the uploading of model information(parameters) might lead to the leak of privacy. Here I will introduce three methods of commonly used privacy protection in Federated Learning.

### Model Aggregation

Each participant will train a local model using its own data and then share only the updates to the model parameters, rather than the raw data itself, with a central server. The central server aggregates these updates to produce a more accurate and refined global model.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-09-08-031649.png" alt="Electronics | Free Full-Text | Reviewing Federated Learning Aggregation  Algorithms; Strategies, Contributions, Limitations and Future Perspectives" style="zoom:50%;" />

Model aggregation is a crucial step in federated learning because it allows information from different sources to be combined while preserving data privacy and security. 

Based on the documentation of TensorFlow, there are always at least two layers of aggregation in federated learning: local on-device aggregation, and cross-device (or federated) aggregation.

- **local on-device aggregation**: This level of aggregation refers to aggregation across multiple batches of examples owned by an individual client. 
- **cross-device (or federated) aggregation**: This level of aggregation refers to aggregation across multiple clients (devices) in the system.

The choice of aggregation method depends on the specific requirements and constraints of the federated learning application, including privacy concerns, communication bandwidth limitations, and the need for robustness.

### Homomorphic encryption

In homomorphic encryption, users must hold the key to get the information of the original data. More importantly, users can calculate and process the encrypted data, but no original data will be disclosed in the process, allowing performing computations on the encrypted data without decrypting it.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-09-08-032727.png" alt="image-20230907202727662" style="zoom:80%;" />

Homomorphic Encryption allows participants to encode their local model updates in such a way that even when these updates are sent to the central aggregator, they remain completely unintelligible. This process can be divided into four steps:

1. **Homomorphic Encryption**: Each participant applies homomorphic encryption to their local model updates before sharing them.
2. **Secure Aggregation**: The central aggregator only receives these homomorphically encrypted model updates. It cannot decipher or gain any insights from the updates.
3. **Encrypted Aggregated Model**: The aggregator combines these encrypted model updates to create an aggregated model, and this model remains encrypted throughout the process.
4. **Decryption and Iteration**: The encrypted aggregated model is then sent back to the participants. Each participant has the decryption key to unlock this model, revealing the improved global model. The participants can then continue with the next round of training using this updated model.

### Differential privacy

Differential privacy is proposed by Dwork in 2006 to solve the problem of privacy leaking in database. 

> The calculation results of the database are insensitive to the changes of a specific record, and a single record in the dataset or not in the dataset has little impact on the calculation results. Therefore, the risk of privacy disclosure caused by the addition of a record to the dataset is controlled in a very small and acceptable range, and the attacker cannot obtain accurate individual information by observing the calculation results.

![基于差分隐私联邦学习的系统模型。](https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-09-08-042022.jpg)

A traditional method is to add noise to the output to apply differential privacy in the process of gradient iteration, so as to achieve the goal of protecting user privacy. However, adding noise to the model will inevitably decrease the validity of the result. A lot of research work is carried out around the two aspects of privacy protection and validity to achieve the balance between privacy and validity.

The TensorFlow provides users with interface of differential privacy on application of EMINST dataset, using the method of adaptive clipping method of [Andrew et al. 2021, Differentially Private Learning with Adaptive Clipping](https://arxiv.org/abs/1905.03871) to avoid adding unnecessary noise and reduce the penetration from the adding noise to the model.

### References

[1] [Federated Learning](https://en.wikipedia.org/wiki/Federated_learning)

[2] Zhang C, Xie Y, Bai H, et al. A survey on federated learning[J]. Knowledge-Based Systems, 2021, 216: 106775.

[3] Yang Q, Liu Y, Chen T, et al. Federated machine learning: Concept and applications[J]. ACM Transactions on Intelligent Systems and Technology (TIST), 2019, 10(2): 1-19.

[4] [Federated Learning in Tensorflow](https://www.tensorflow.org/federated/federated_learning)

[5] Moshawrab M, Adda M, Bouzouane A, et al. Reviewing Federated Learning Aggregation Algorithms; Strategies, Contributions, Limitations and Future Perspectives[J]. Electronics, 2023, 12(10): 2287.

[6] [IBM Federated Learning with homomorphic Encryption](https://medium.com/ibm-data-ai/ibm-federated-learning-with-homomorphic-encryption-a4cad23f012c)

[7] Dwork C. Differential privacy: A survey of results[C]//International conference on theory and applications of models of computation. Berlin, Heidelberg: Springer Berlin Heidelberg, 2008: 1-19.

[8] Abadi M, Chu A, Goodfellow I, et al. Deep learning with differential privacy[C]//Proceedings of the 2016 ACM SIGSAC conference on computer and communications security. 2016: 308-318.

[9] Andrew G, Thakkar O, McMahan B, et al. Differentially private learning with adaptive clipping[J]. Advances in Neural Information Processing Systems, 2021, 34: 17455-17466.

