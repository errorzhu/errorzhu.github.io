# prompt

# 参数

1. temperature
   
   温度控制令牌选择的随机程度。较低的温度适用于期望更确定性响应的提示，而较高的温度可能会导致更多样化或意外的结果。温度为 0（贪婪解码）是确定性的：始终选择最高概率的令牌（但请注意，如果两个令牌具有相同的最高预测概率，则根据 tiebreaking 的实现方式，您可能并不总是在温度为 0 时获得相同的输出

2. Top-K 
   
   抽样从模型的预测分布中选择前 K 个最可能的代币。top-K 越高，模型的输出就越有创意和多样化;top-K 越低，模型的输出就越不安分和真实。top-K 为 1 等效于贪婪解码。

3. Top-P 
   
   抽样选择累积概率不超过特定值 （P） 的前 TOKEN。P 的值范围从 0（贪婪解码）到 1（LLM 词汇表中的所有标记）

# 技术

1. 0样本提示

2. 少样本提示
   
   ```
   Parse a customer's pizza order into valid JSON:
   EXAMPLE:
   I want a small pizza with cheese, tomato sauce, and pepperoni.
   JSON Response:
   ```
   {
   "size": "small",
   "type": "normal",
   "ingredients": [["cheese", "tomato sauce", "peperoni"]]
   }
   ```
   EXAMPLE:
   Can I get a large pizza with tomato sauce, basil and mozzarella 
   {
   "size": "large",
   "type": "normal",
   "ingredients": [["tomato sauce", "bazel", "mozzarella"]]
   }
   Now, I would like a large pizza, with the first half cheese and 
   mozzarella. And the other tomato sauce, ham and pineapple.
   JSON Response:
   ```
   
   

3. system,contextual,role 提示
   
   System prompting
   
   设置语言模型的整体上下文和目的。它定义了模型应该做什么的“大局”，比如翻译语言、对评论进行分类等
   
   Contextual prompting
   
   提供与当前对话或任务相关的具体详细信息或背景信息。它有助于模型了解所询问内容的细微差别，并相应地定制响应
   
   Role prompting
   
   为语言模型分配要采用的特定字符或身份。这有助于模型生成与分配的角色及其相关知识和行为一致的响应

4.  回退式提示
   
   ```
   Based on popular first-person shooter action games, what are 
   5 fictional key settings that contribute to a challenging and 
   engaging level storyline in a first-person shooter video game?
   ```
   
   ```
   Context: 5 engaging themes for a first person shooter video game:
   1. **Abandoned Military Base**: A sprawling, post-apocalyptic 
   military complex crawling with mutated soldiers and rogue 
   robots, ideal for challenging firearm combat.
   2. **Cyberpunk City**: A neon-lit, futuristic urban environment 
   with towering skyscrapers and dense alleyways, featuring 
   cybernetically enhanced enemies and hacking mechanics.
   3. **Alien Spaceship**: A vast alien vessel stranded on 
   Earth, with eerie corridors, zero-gravity sections, and 
   extraterrestrial creatures to encounter.
   4. **Zombie-Infested Town**: A desolate town overrun by hordes of 
   aggressive zombies, featuring intense close-quarters combat and 
   puzzle-solving to find safe passage.
   5. **Underwater Research Facility**: A deep-sea laboratory flooded 
   with water, filled with mutated aquatic creatures, and requiring 
   stealth and underwater exploration skills to survive.
   Take one of the themes and write a one paragraph storyline 
   for a new level of a first-person shooter video game that is 
   challenging and engaging.
   ```

5.  思维链
   
   ```
   When I was 3 years old, my partner was 3 times my age. Now, 
   I am 20 years old. How old is my partner? Let's think step 
   by step
   ```
   
   ```
   Q: When my brother was 2 years old, I was double his age. Now 
   I am 40 years old. How old is my brother? Let's think step 
   by step.
   A: When my brother was 2 years, I was 2 * 2 = 4 years old. 
   That's an age difference of 2 years and I am older. Now I am 40 
   years old, so my brother is 40 - 2 = 38 years old. The answer 
   is 38.
   Q: When I was 3 years old, my partner was 3 times my age. Now, 
   I am 20 years old. How old is my partner? Let's think step 
   by step.
   A
   ```
   
   

6.  自我一致性
   
   它遵循以下步骤：1. 生成不同的推理路径：LLM 多次提供相同的提示。高温设置鼓励模型生成不同的推理路径和对问题的看法。2. 从每个生成的响应中提取答案。3. 选择最常见的答案。
   
   ```
   EMAIL:
   ```
   Hi,
   I have seen you use Wordpress for your website. A great open 
   source content management system. I have used it in the past 
   too. It comes with lots of great user plugins. And it's pretty 
   easy to set up.
   I did notice a bug in the contact form, which happens when 
   you select the name field. See the attached screenshot of me 
   entering text in the name field. Notice the JavaScript alert 
   box that I inv0k3d.
   But for the rest it's a great website. I enjoy reading it. Feel 
   free to leave the bug in the website, because it gives me more 
   interesting things to read.
   Cheers,
   Harry the Hacker.
   ```
   Classify the above email as IMPORTANT or NOT IMPORTANT. Let's 
   think step by step and explain why.
   
   ```
   
   

7. 思维树

8. ReAct
   
   ```
   Answer the following questions as best you can. You have access to the following tools:
   
   How many kids do the band members of Metallica have?
   Use the following format:
   
   Question: the input question you must answer
   Thought: you should always think about what to do
   Action: the action to take, should be one of [{tool_names}]
   Action Input: the input to the action
   Observation: the result of the action
   ... (this Thought/Action/Action Input/Observation can be repeated zero or more times)
   Thought: I now know the final answer
   Final Answer: the final answer to the original input question
   
   Begin!
   
   
   ```
   
   

9.  自动提示工程
   
   ```
   We have a band merchandise t-shirt webshop, and to train a 
   chatbot we need various ways to order: "One Metallica t-shirt 
   size S". Generate 10 variants, with the same semantics but keep 
   the same meaning
   ```
   
   编写将生成输出变体的提示
   
   通过根据所选指标对候选指令进行评分来评估所有候选指令，例如，您可以使用。BLEU （双语评估替补） 或 ROUGE （用于 Gisting 评估的以回忆为导向的替补）
   
   选择评估分数最高的候选指令。此候选者将是您可以在软件应用程序或聊天机器人中使用的最终提示。您还可以调整选择提示并再次评估

10. 

# 参考资料

https://github.com/NirDiamant/Prompt_Engineering

https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools


