---
layout: post
title: evaluate(input_sentence) 
tags: [lua文章]
categories: [topic]
---
详细的记录 evaluate函数的实现。  
解决报错

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    

|

    
    
    ValueError                                Traceback (most recent call last)  
    <ipython-input-44-2ec1176683f0> in <module>  
    ----> 1 translate(u'Estoy trabajando.')  
      
    <ipython-input-43-4364cc5c7981> in translate(input_sentence)  
         49   
         50 def translate(input_sentence):  
    ---> 51     results, input_sentence, attention_matrix = evaluate(input_sentence)  
         52   
         53     print("Input: %s" % (input_sentence))  
      
    <ipython-input-43-4364cc5c7981> in evaluate(input_sentence)  
         20     decoding_input = tf.expand_dims([out_tokenizer.word_index['<start>']], 0)  
         21     for t in range(max_length_output):  
    ---> 22         predictions. decoding_hidden, attention_weights = decoder(decoding_input, decoding_hidden, encoding_outputs)  
         23         attention_weights = tf.reshape(attention_weights, (-1,))  
         24         attention_matrix[t] = attention_weights.numpy()  
      
    ValueError: too many values to unpack (expected 2)  
      
  
---|---  
  
### 注意看predictions 后面的标点符号

接收的是一个文本的输入，首先就要转换成适合模型的数据类型。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    54  
    55  
    56  
    57  
    

|

    
    
    def evaluate(input_sentence):  
        attention_matrix = np.zeros((max_length_output, max_length_input))   
        input_sentence = preprocess_sentence(input_sentence) # 输入的句子进行预处理。就是分割标点符号/  
      
        inputs = [input_tokenizer.word_index[token] for token in input_sentence.split(' ')] # text--->id 把句子转换成id  
        inputs = keras.preprocessing.sequence.pad_sequences([inputs], maxlen = max_length_input, padding= 'post') # 把转换成id的向量，进行padding  
        inputs = tf.convert_to_tensor(inputs) #把向量转换为tensor  
      
        results = '' # 定义str, 保存translate的结果  
      
    #     encoding_hidden = encoder.initialize_hidden_state()  
      
        encoding_hidden = tf.zeros((1, units)) #初始化encoding_hidden层  
      
        encoding_outputs, encoding_hidden = encoder(inputs, encoding_hidden) # 这一步得到的encoding_hidden就是decoding_hidden 的第一个值  
        decoding_hidden = encoding_hidden  
      
      
        decoding_input = tf.expand_dims([out_tokenizer.word_index['<start>']], 0) # 找到开始的第一个输入的id  
        for t in range(max_length_output):  
            predictions, decoding_hidden, attention_weights = decoder(decoding_input, decoding_hidden, encoding_outputs)  
            attention_weights = tf.reshape(attention_weights, (-1,))  
            attention_matrix[t] = attention_weights.numpy()  
      
            predicted_id = tf.argmax(predictions[0]).numpy()  
      
            results += out_tokenizer.index_word[predicted_id] + ' '  
      
            if out_tokenizer.index_word[predicted_id] == '<end>':  
                return results, input_sentence, attention_matrix  
      
            decoding_input = tf.expand_dims([predicted_id], 0)  
        return results, input_sentence, attention_matrix  
      
    def plot_attention(attention_matrix, input_sentence, predicted_sentence):  
        fig = plt.figure(figsize=(10,10))  
        ax = fig.add_subplot(1, 1, 1)  
      
        ax.matshow(attention_matrix, cmap='viridis')  
      
        font_dict = {'fontsize': 14}  
      
        ax.set_xticklabels([''] + input_sentence,  
                                  fontdict = font_dict, rotation = 90)  
        ax.sey_yticklables([''] + predicted_sentence,  
                                  fontdict = font_dict,)  
        plt.show()  
      
    def translate(input_sentence):  
        results, input_sentence, attention_matrix = evaluate(input_sentence)  
      
        print("Input: %s" % (input_sentence))  
        print("Predicted translation: %s" % (results))  
      
        attention_matrix = attention_matrix[:len(results.split(' ')),  
                                                           :len(input_sentence.split(' '))]  
        plot_attention(attention_matrix, input_sentence.split(' '), results.split(' '))  
      
  
---|---