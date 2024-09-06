---
layout: post
title:  "Generate names with AI: n-grams"
date:   2023-05-20 08:00:00 +0000
categories: [C++]
---

You and your friend find yourselves rocking out in the garage, jamming to your favorite tunes, and feeling the exhilaration of creating music together. As the energy and creativity flow, you realize that you're missing one crucial element—an awesome band name that captures the essence of your musical chemistry. But then you recall that your friend has recently developed a cool C++ application that can generate band names based on the first 6000 most popular artists from MTV. Excitement fills the air as you decide to fire up the application and unleash its creative power to give birth to a unique and captivating band name that perfectly represents your musical style.


With the AI's suggestion of "The Woogeles" as the band name, you're instantly captivated by its uniqueness and charm. Intrigued by how the C++ application generated such a fitting name, you turn to your friend and ask about the inner workings of the program.

Your friend explains that the C++ application utilizes the power of n-grams, a statistical language modeling technique. The program is designed to analyze a corpus of text, in this case, the names of the first 6000 most popular artists from MTV. By breaking down the artist names into n-grams (contiguous sequences of characters), the application creates a map that associates each n-gram with the characters that typically follow it.

When you request a band name, the program starts with a randomly selected n-gram from the corpus. It then uses the n-grams map to probabilistically determine the next character based on the previous n-gram. This process continues until a sentinel end value is reached, resulting in a unique and plausible name.

The AI's ability to suggest "The Woogeles" stems from its analysis of patterns and statistical regularities within the artist name corpus. By leveraging these patterns, the AI is able to generate band names that feel familiar yet distinctly original.

Amazed by your friend's application and its intelligent use of n-grams, you're filled with excitement and anticipation for the musical journey ahead with "The Woogeles" as your band's captivating identity.

Your friend pulls up his github [page](https://github.com/NikBomb/makemore_cpp/blob/master/makemore.cpp) and shows you this code that demonstrates the generation of a name using n-grams. He goes through it and explains each section

```cpp
const uint16_t n_size = 3;
```
n_size: here we define the size of the n-grams used for name generation.

```cpp
    if (database.is_open()){
        std::string buffer;
        while(database){
            std::getline(database, buffer);
            if (buffer != "") {
                for (const char c : buffer){
                    alphabet.insert(c);
                }
                names.push_back(std::move(buffer));
            }
        }
    }

    alphabet.insert('.');
```

In this section we read the artist database file (artists_dbase.txt) and store each artist name in the names vector.
We also creates a set called alphabet to store all the unique characters encountered in the artist names.


```cpp
   /*Create n_grams*/
    std::map<std::string, uint64_t> n_grams;
    const std::string start_stop = std::string(n_size - 1, '/');
    for(auto& name : names){
        name = start_stop + name + start_stop; 
        for (size_t i = 0; i <= name.size() - n_size; ++i) {   
            auto result = n_grams.find(name.substr(i, n_size));
            if(result == n_grams.end()){
                n_grams.insert(std::make_pair(name.substr(i, n_size), 1));
            } else {
                result->second += 1;
            }
        }
    }
```

Then we need to create the n-grams from the artist names.
The n-grams are formed by appending special characters (start_stop) to the beginning and end of each artist name, and then extracting subsequences of length n_size. The map n_grams associates each n-gram with the number of times it occurs in the database.


```cpp
 auto name = std::string(n_size - 1, '/');
    std::vector<double> probs;
    std::unordered_map<uint64_t, std::string> mapPos;
    std::random_device rd;
    std::mt19937 gen(rd());

    size_t win_start = -1;
    while (name[name.size() - 1] != '/' || win_start == -1){
        win_start++;
        auto window = name.substr(win_start, n_size - 1);
       
        
        for (const auto c : alphabet){
        auto freq = n_grams.find(window + c);
            if (freq != n_grams.end()) {
                mapPos[mapPos.size()] = c;
                probs.push_back(freq->second);
            }
        }
        std::discrete_distribution<size_t> distribution(probs.begin(), probs.end());
        auto n = distribution(gen);
        name = name + mapPos[n];
        mapPos.clear();
        probs.clear();
        distribution.reset();
    }    

    name = name.substr(n_size - 1, name.size() - n_size);
    std::cout << "My generated name is: " <<  name << std::endl;
```

Finally we are ready to generate a name, based on the n-grams.
We initalize the name with the special character (start_stop). Then we use a sliding window approach to select the appropriate next character based on the n-grams. 
At each iteration we append a character from the alphabet based on their frequencies in the n-grams.
The randomization is achieved using the std::discrete_distribution function, which takes the frequencies as probabilities.
The loop continues until the generated name ends with the special character (start_stop).

Filled with confidence, because you know now about n-grams, you walk in the sun and smile because "The Woogeles" will be soon on the top of that database used by your friend, Nico.

I hope you found this post exhilarating, informative and fun! See you next time.