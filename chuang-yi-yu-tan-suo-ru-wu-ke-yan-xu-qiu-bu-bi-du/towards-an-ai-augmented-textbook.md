---
description: LearnLM Team, Google
---

# Towards an AI-Augmented Textbook

## Towards an AI-Augmented Textbook

LearnLM Team, Google

Textbooks are a cornerstone of education, but they have a fundamental limitation: they are a one-sizefits-all medium. Any new material or alternative representation requires arduous human effort, so that textbooks cannot be adapted in a scalable manner. We present an approach for transforming and augmenting textbooks using generative AI, adding layers of multiple representations and personalization while maintaining content integrity and quality. We refer to the system built with this approach as Learn Your Way. We report pedagogical evaluations of the different transformations and augmentations, and present the results of a a randomized control trial, highlighting the advantages of learning with Learn Your Way over regular textbook usage.

Keywords: Personalized learning, generative education, content transformations

### 1. Introduction

Recent advances in generative Artificial Intelligence (Gen-AI) have the potential to revolutionize education, but this potential is yet to be realized in full. It requires a responsible, multidisciplinary approach to weave together learning science and cutting edge technology. In this work, we focus on a central aspect of the current learning journey: exploring textbook material. Traditionally, every school selects several textbooks that are meant for use by all learners. The textbooks, by definition, are inflexible and not adaptive, as it is impractical to manually create a version for every audience, and certainly not one that would adapt to individual user needs. Here we argue that in the age of Gen-AI, this notion of a flexible and personalized textbook is in fact within reach. Specifically, we show how textbooks can be transformed into a richer and more personalized form, while maintaining the integrity of the original content, and adding layers that promote effective learning.

Our textbook augmentation approach takes as input a textbook segment or chapters and uses them as the basis for extensive generated content, practice and evaluation. Our approach rests on two key concepts that underlie the corresponding augmentations of the original content: multiple representations and personalization. We propose a two step AI generation scheme whereby the original text is first personalized, and then transformed into a range of presentation forms and assessment components. A key desiderata in this process is that content is adequately aligned with the source and curriculum, and that the presentation is engaging and pedagogically effective. We implement our approach in an experimental learning experience that we call Learn Your Way.

We begin with the pedagogical observation that learning can be more effective when the experience is adapted to the characteristics and needs of the learner \[see 1,2, for a review of personalization approaches]. Learn Your Way is thus designed to first re-generate the original textbook content, based on specific learner attributes. In addition, Learn Your Way generates assessment opportunities for the learner which serve to create a signal about their progress, reflect personalized feedback to the learner, and influence subsequent steps.

The value of multiple representations of content has been studied in learning science \[e.g., see 3]. For example, dual coding theory \[4] suggests that multiple representations have the advantage of forging links between different encodings of the same concepts, thus reinforcing the corresponding mental conceptual structures. Learn Your Way is therefore augmented with multiple views of the

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876468961.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=aHsb3F1ZAoejFmDB70LYdOooQy0%3D\&Expires=1776481269)

Figure 1 | An example of the Learn Your Way learning experience. Centerpiece is the "Immersive Text" view, that shows the source material (OpenStax's Disruptions in the Immune System content) transformed to 6th grade level and adapted for a personal interest in gaming. The Immersive Text contains various generative add-ons such as personalized examples, embedded questions, and more. At any given time, the learner can also switch to alternative views of the entire material such as narrated slides or an audio lesson, which are also personalized.

material (audio lessons, narrated slides, and mind maps), which learners can interact with and choose from. Providing these options is in line with the view that personalized systems should also be adaptable, offering learners agency to decide on their learning path \[5-7]. It is also motivated by theories of self-regulated learning (SRL) \[8] and visible learning \[9] that acknowledge the importance of supporting learners with cognate control of their learning process.

Figure 1 shows the central Learn Your Way view, demonstrating how personalization and multiple representations come together. The resulting AI-augmented textbook provides the learner with a personalized and engaging learning experience, while also allowing them to choose from different modalities in order to enhance understanding. In the next sections we describe each of the components in Learn Your Way, along with an evaluation of the pedagogical merit of each one. Finally, we report the results of a randomized controlled study showing that learning with our personalization and multiple representations Learn Your Way system can improve learning efficacy compared to a standard Digital Reader over the same material. Taken together, our results demonstrate the potential to re-imagine the medium of a textbook in the age of generative AI.

### 2. Textbook Augmentation via Personalization and Multiple-Views

We assume source-of-truth material which is defined by the learner's curriculum and learning goals. For simplicity, think of a section in a textbook, although the source-of-truth can be a more complex collection of knowledge and skills to be delivered. Our goal is to explore how transforming the source material can increase content engagement and efficacy. Gen-AI offers four key opportunities in this context. First, it can generate such content for any material the learner is interested in. Second, it can do so while adapting to the specific attributes and needs of the learner. This is in contrast to the generation of personalized learning material by human educators, which is a much longer process and is impractical to do at scale. Third, AI can be used to generate different representations of the material, including visualizations and audio-based formats, which are known to further enhance the efficacy of learning \[3,4]. Finally, AI can generate formative assessment elements tailored to the learner, allowing them to monitor and regulate progress. As \[10] notes, formative assessment is a critical driver of learning, and in particular self-regulated learning.

Our textbook transformation and augmentation follows a two step approach. In the "Text Personalization" stage, we rewrite the material to match specific personal attributes of the learner. Then in the "Content Transformations" stage, we create multiple views of the rewritten material. These allow the user to choose their own learning path, interleaving complementary representations of the same conceptual structures. Figure 2 demonstrates this process, showing two different personalization transformations, that results in different views. Unless otherwise noted, all transformations and augmentations described below rely directly on Gemini 2.5 Pro, a leading model for education \[11], without additional fine-tuning.

### 2.1. Text Personalization

As explained above, our approach first transforms the original text into a more personalized form. A key choice in this process is what specific attributes of the learner should be personalized to. For simplicity, we focus on two key attributes: grade-level and personal interests. There are of course many additional attributes to consider on the path towards more comprehensive personalization.

### Personalization to Grade Level.

Adaptation of the material to match the reading grade level of the learner is a core transformation that provides the basis for all other transformations that follow. The text is generatively adapted, with the goal of matching the Flesch-Kincaid Grade (FKG) \[12, 13] for that level, while maintaining factuality and coverage of the material. This is known as re-leveling and is part of Gemini 2.5 Pro core education capabilities. See \[11], Section 2.2 for an evaluation.

### Personalization to Interests.

The Learn Your Way experience asks the learners, in addition to the grade level, for their personal interests. Currently, for simplicity, the learner is asked to select one of several common interests (e.g., sports, music, food). This information is then used to rewrite the original text, making it more relatable. This also serves the purpose of mapping new knowledge to existing conceptual networks used by the learners, thus making learning more effective. As \[14] notes: "individuals' existing knowledge serves as a base for subsequent learning and performance". Their review further argues that "prior knowledge guides readers' comprehension of written language". Our Gen-AI rewriting is done in a focused manner, by first selecting parts of the text that are particularly amenable to personalization, and then replacing only these parts with an AI-rewritten personalized version. This has the added advantage of highlighting the personalized text, thus informing the learner that it has been specialized to their interests. See example in Figure 2 for Newton's third law example, rewritten for two different interests: basketball and art.

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469024.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=fHvNbzgv3l7ILpE7hb8cXFBmt3w%3D\&Expires=1776481269)

Figure 2 | An illustration of the two step generation procedure used in Learn Your Way. Here, a generic example from OpenStax's Newton's Third Law of Motion is first personalized for the personal interest of 'basketball' (left) and 'art' (right), and then expanded into different presentation formats.

### 2.2. Content Transformations

The rewriting phase above results in text that is adapted to the learner. We use this text as the basis for multiple content transformations, each providing a different view of the material. Since all are based on the personalized text in Section 2.1, they are also similarly personalized. Below we provide a description of the different transformations. We describe the Slides, Audio and Mind maps transformations below, and the "Immersive Text" transformation in Section 2.2.1.

### Slides and Narration.

Learners often benefit from a class-like slide sequence that covers the core material in brief, while also suggesting interest-capturing questions that precede the material, and activities aimed at engagement.

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469039.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=JdxJsxzvK6ACa2nGU6FAyyoHPCQ%3D\&Expires=1776481269)

Figure 3 | Example slide in the deck generated for OpenStax's How To Organize Economies source and adapted to the learner interest in 'soccer'.

This provides an alternative presentation format that could be more effective for some learners.The fact that the slides are based on personalized text makes them further effective. Figure 3 shows a sample slide with an adaptation of an example of market economy to the soccer domain. The Learn Your Way experience also provides an additional optional generated narration for the slides. The narration is meant to resemble a recorded lesson, and the narrated text is not restricted to the text in the slides, but is rather designed to be natural and complementary to the slides.

### Audio-Graphic lesson.

This transformation aims at a comprehensive and detailed coverage of the material, delivered in a audio-graphic form that simulates a conversation between a teacher and a student about the material. To allow for a realistic experience, the teacher and student turns are generated iteratively using independent Gemini "personas". This allows for a realistic experience where the (virtual) student does not see the material before it is presented and may, for example, respond to questions with answers that are not part of the original material and uncover common learner misconceptions. In addition to the audio conversation, the lesson contains a graphical representation of the key concepts and the relationships between them, which is dynamically presented to the learner. This combination of audio and visual components is motivated by dual coding theory, which suggests that multiple representations of concepts serve to strengthen the corresponding mental conceptual structures.

### Mind Maps.

This common graphical representation organizes information hierarchically, and allows for a broad view of the material at different levels of granularity. It is often useful as a mechanism for organizing the material following a detailed learning session, or as an organizational reminder of the entire source material. We annotate the map nodes with illustrative texts and images derived from the source, and allow the user to expand and collapse nodes, in effect zooming in and out of the conceptual hierarchy. See Figure 4 for an example.

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469048.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=83u4XfkwsKhNE1Oht9Cy4WiotOI%3D\&Expires=1776481269)

Figure 4 | An example mind map created for OpenStax's Early Human Evolution and Migration source material. Different nodes can be expanded to gain a more granular view of the material, with leaf nodes annotated with text or relevant visuals.

### 2.2.1. Immersive Text

The above transformations, as effective as they may be, do not stand alone. They are intended to supplement a coherent and comprehensive text articulating the content of the source in full detail. Such a text can be enhanced by interleaving it with personalized elements and multiple modalities. We refer to this as an "Immersive Text" (see Figure 1). After every section of the text we optionally include several added components that are meant to enhance the learning experience, as detailed below. We also include assessment components, as described in Section 3.

### Timeline.

Source material often contains sequences, such as a series of events in history or the stages of an experiment or algorithmic approach. "Timelines" can convey these sequences visually, reducing cognitive load and making it easier for the learner to follow the details. To generate these, the source material is first scanned to identify candidate sequences, followed by the generation of the timeline and appropriate placement within the material. Such transformations also offer a chance for interactive practice that is closely tied to the material: the learner is simply asked to drag-and-drop boxes into their appropriate place in the sequence.

### Memory Aid.

Learning new material often involves memorizing facts, a task which can be challenging for learners. There are many strategies that can help learners address this need. We focus on the common strategy of mnemonics, a memorization approach where each item to remember is associated with a word that begins with the same first letter, and the sequence of words forms a sentence. With on-the-fly generation we are no longer restricted to commonly used mnemonics whose coverage is scarce. Instead, given the input material, Gemini is used to first identify elements in the material that are hard to memorize. Then a mnemonic is generated with two requirements in addition to the constraint of being a valid mnemonic: form a coherent and easy to remember sentence, and form a sentence that has close semantic association with the material to be remembered.

### Visual Illustrations.

Visual learning is broadly recognized as a powerful medium, and many textbooks include explanatory diagrams and drawings. It is natural to use AI image generation tools to produce such visuals. However, our initial exploration found that even the most advanced AI image generation models struggle to

A market is a system connecting buyers and sellers. The international soccer transfer market is a prime example, where clubs act as buyers and sellers of players. In this market, decisions are decentralized, and a player's income (value) is determined by how much society (clubs) values their skills.

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469056.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=bdCqVPEIvBmAkNQD5KyPcKJA86I%3D\&Expires=1776481269)

Figure 5 | An example of personalized visual generation that captures a key concept in an engaging manner that is based on the interest of the learner in soccer for OpenStax's How to Organize Economies material. The personalized text next to the image shows that the original text has been adapted to the topic of sports. The image is a visual illustration of the text, and thus is also personalized.

produce these types of images. This can be explained by the fact that such models are trained to produce realistic images that are high on detail, and sometimes return inaccurate images when asked for 'simple' or 'educational' ones. $ ^{1} $ To overcome this, we fine-tuned a model specifically for this task. This model is applied to parts of the material that Gemini identifies as worthy of illustration. An example, is shown in Figure 5.

### 3. Practice and Assessment

Formative assessment is arguably one of the primary drivers of learning \[15], and augments the multiple views of material discussed previously. Indeed, effective while-you-learn assessment and on-the-fly feedback can help reinforce concepts and increase knowledge and skill retention \[e.g., see 16]. We therefore augment Learn Your Way with two assessment components, described below. Both components appear within the Immersive Text view.

### Embedded Questions.

Embedded questions are dynamically generated questions that are grounded and associated with specific segments of the source material. These questions serve to convert the reading experience from passive to active and to keep the learner engaged by providing immediate feedback. They also reinforce the concept being learned. In Learn Your Way they are presented as multiple choice questions that appear when the learner clicks a question mark in the Immersive Text view. See Figure 6.

### Quizzes.

Section-level quizzes aim at deeper understanding once a section has been read and assimilated. The quizzes are dynamically generated and grounded to all of the material in the section. They consist of 5-10 multiple choice questions of various difficulties and types. At the end of the quiz, an overall assessment is provided that includes both a numerical score as well as targeted feedback that highlights strengths (or Glows) and areas for improvement (or Grows).

Figure 6 | An example of an embedded question for OpenStax's topic of How to Organize Economies.

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469063.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=wVk42Xc%2BDdfExKBpRC9zavuFI3o%3D\&Expires=1776481269)

### 4. Pedagogical Evaluations

In order to assess the quality of the different augmentation and transformation components used in Learn Your Way, we asked multiple pedagogical experts to evaluate the quality of each according to a pedagogical rubric over several criteria.

For the evaluation we used ten source-of-truth PDFs, ranging in topics from sociology to physics from OpenStax (see full list in Appendix A). For grade level personalization, we considered three grade levels (7th grade, 10th grade and undergraduate level) and three personal interests (basketball, music and food). Each PDF was assigned three random combinations of grade level and personal interests (out of the nine possible combinations). Each of the configurations above was then provided as input to Learn Your Way, which generated the transformations and assessments described in Section 2.2 and Section 3. The experts were asked to evaluate the overall merit of the Learn Your Way experience, as well as the individual components of the system:

* Content transformations: Slides, Narrated Slides, Audio Lesson, Mind map.
* Immersive text components: Timeline, Memory Aid, Visual illustrations.
* Personalization to Interests: as noted in Section 2.1, parts of the original text were replaced with a version personalized to the interests of the user. These parts were also highlighted, and thus we could ask the pedagogical raters to evaluate them.
* Assessment: as noted in Section 3, the immersive text contains two types of assessments: embedded questions and quizzes. We evaluate these separately.

We asked the pedagogical experts to evaluate the quality of each component with respect to several basic criteria such as coverage, as well as criteria that capture key learning science principles. See Appendix B for the evaluation rubrics for all criteria. For example, for the Accuracy criterion, they were asked if the component is "faithful to the source and accurate". The criteria were Coverage, Emphasis, Cognitive load, Active learning, Deepen meta-cognition, Motivation & curiosity, Adaptability, and Clarity of learning intentions. For each of these, following the guidance of pedagogical rubrics (see Appendix B), raters provided an agree (1.0), neutral/partial (0.5), or disagree (0.0) rating. Finally, each component was evaluated by three different raters.

The results are summarized in Figure 7, which shows that all components have relatively high pedagogy values and the overall experience is rated over 0.90 across all axes. The component with the lowest scores is that of Visual Illustration. This is to be expected given the difficult of generating high quality pedagogical images.

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469073.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=0g%2BtpZL1Fpv2OovHmvbiH9%2FVGqI%3D\&Expires=1776481269)

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_2_1775876469081.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=LL%2F%2F%2BjANbdB8TOsvEeh%2BzsbwHes%3D\&Expires=1776481269)

Figure 7 | Rating of the various components that make up Learn Your Way, as rated by pedagogy experts. Top figure shows high-level metrics, and the bottom figure shows additional metrics that arise from core learning science principles. Rating is based on the rubrics detailed in Appendix B. Shown is the average across the three raters across all source materials.

A closer inspection of the results presented in Figure 7 reveals additional more nuanced insights. For example, the slides format received the lowest 'engagement' score of all capabilities. On the other hand, these same slides but with generative narration received a significantly higher score. This is in line with the fact that slides are often presented alongside narration, and thus this combination is more engaging for learners. These types of insights are invaluable for improving future versions of all Learn Your Way components.

### 5. Efficacy Study Design and Results

Above we reported on a pedagogical evaluation of the different generated components for multiple topics and personalizations. However, the impact is much greater when the various capabilities come together in a learning experience, and it is therefore important to measure their holistic pedagogical value in terms of learning efficacy. To evaluate Learn Your Way from this perspective, we ran an experiment where students had to learn an unfamiliar textbook chapter.

### Student Recruitment.

Sixty students, aged 15-18 years old, were recruited from the Chicago area across urban, suburban, and rural schools. To ensure that participating students had similar reading comprehension skills, we gave them a reading comprehension and assessment task as part of the recruitment criteria. The assessment included a short passage followed by a series of questions about the passage. As such, the

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469112.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=Jv1VXw7saQBz1ebWDRRYldtuF50%3D\&Expires=1776481269)

Figure 8 | Overview of the experimental setup of the personalized learning efficacy study.

format of this recruitment task was similar to the experiment, but with different learning material (it was a text on ocean waves). The average score on the assessment was 6.4, with a standard deviation of 2.3. We included students who scored 1 standard deviation above or below the mean, which corresponded to receiving a score of 4-9 out of 10 on the assessment.

### Learning Material and Assessment.

In our study, all students were presented with the same textbook chapter (Brain Development for Adolescents from LibreTexts). In choosing the material, a panel of pedagogy experts was asked to find a text that conformed to several pedagogical criteria, including length, familiarity, and interest. To account for pre-study familiarity with the topic in our analysis, students were asked to record their familiarity with the topic before the study on a scale of 1-5 with 5 corresponding to very familiar.

After exploring the learning material, all students received the same assessments, which were written by a pedagogical expert. Two assessments were written by a pedagogical expert to evaluate understanding of the learning material. The first "Immediate Assessment" was a 15 minute assessment designed to be taken immediately after the learning session. This included short answer questions, single-answer, multiple choice questions, matching questions, multi-answer, and multiple choice questions. The questions were written by the expert to match Bloom's taxonomy level expected at that age group. Three days post-session, participants were emailed a follow-up "Retention Assessment" that measured long-term recall of the content they had learned during the experiment. It was designed to take 5-10 minutes, and included a short answer question, a single-answer multiple choice question, and a matching question. Participants were given four days to complete it. 58 out of 60 participants completed the retention assessment.

Immediately after the first assessment, students were also asked to rate its difficulty. Students using Learn Your Way rated the assessment's difficulty as 3.6 on average (on a scale of 1-5), slightly higher than those in the control group who gave an average rating of 3.37, but the difference is not statistically significant between the groups using the Mann-Whitney U test. These values suggest that, from the learners' perspective, the assessment was neither too difficult nor too easy.

### Experimental Setup.

Participants were brought into the lab and introduced to the study via informed consent, followed by an overview of the research sessions. Each participant was randomly assigned to one of two learning conditions: Learn Your Way and a Digital Reader (Adobe Acrobat Reader v25.001.20531). For students assigned to Learn Your Way, the average reported pre-study familiarity with the topic was $ 2. 6 \pm1. 1 $ . Students who were assigned to the digital reader were slightly more familiar with the topic with an average of $ 2. 9 \pm1. 2 $ , but the difference was not found to be statistically significant using the Mann-Whitney U test.

Each participant had five minutes to review a set of three slides that introduced them to the features available in the tool they were assigned. Participants then used the assigned tool to study the material. Learning time was set to a minimum of 20 minutes and a maximum of 40 minutes. After this time, each participant had 15 minutes to complete the Immediate Assessment via a Qualtrics link.

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_1_1775876469119.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=3ljiLZWfTMvnbYftKjj371c0q5Q%3D\&Expires=1776481269)

![OCR图片](https://maas-watermark-prod-new.cn-wlcb.ufileos.com/ocr%2Fcrop%2F20260411110035cbe036a7d4e141db%2Fcrop_2_1775876469125.png?UCloudPublicKey=TOKEN_6df395df-5d8c-4f69-90f8-a4fe46088958\&Signature=MD0VcTM84TO8%2BK8lc99tg1we18I%3D\&Expires=1776481269)

Figure 9 | (left) Average scores of learners who used Learn Your Way and a Digital Reader for the assessment that immediately followed the learning experience. (right) Average scores for the retention assessment given three days after the learning experience. In both cases, the differences between Learn Your Way and the tool were statistically significant.

Participants did not receive a visible score after completing the assessment. Each participant then had 10 minutes to answer a series of quantitative questions via Qualtrics to share their impressions of the learning tool. The moderator followed-up with a 20 minute 1:1 qualitative interview with each participant, diving deeper into their experience with the tool. Three days after the study, participants received the Retention Assessment, as explained above. The setup is summarized in Figure 8.

### Results.

Figure 9 (left) shows the results from the Immediate Assessment, and Figure 9 (right) shows results from the Retention Assessment. The students who used Learn Your Way received higher scores than those who used the Digital Reader, in both the immediate （ p=0.03 ）and retention （ p=0.03 ） assessments. The Shapiro-Wilk test was used to test normality of the scores and, since this was not the case, the nonparametric Mann-Whitney U test was used to compute statistical significance.

Figure 10 reports results on the learning experience survey. It can be seen that that Learn Your Way provides significant advantages compared to the Digital Reader. Specifically, across all measures that assessed learning experiences, Learn Your Way was consistently evaluated more positively compared to the Digital Reader using the same statistical tests as above.

### 6. Discussion and Future Work

We presented a two stage Gen-AI approach for transforming and augmenting source learning material. Our approach was implemented using Gemini in Learn Your Way, a novel, experimental learning experience. In addition to pedagogical expert evaluation of the quality and merit of the different components of Learn Your Way, a randomized controlled trial substantiated its potential efficacy for real student learning given an unfamiliar textbook chapter.

The scope of our efficacy study has limitations. Since Learn Your Way contains multiple components, including formative quizzes, a natural question is which of these contributes most to learning efficacy. Since the current study did not hone in on this, there might be some transformations that have impact while others do not. It would also be beneficial to broaden the evaluation to multiple textbook chapters and topics. Addressing these questions within a more detailed controlled experiment is possible, but would be costly. On the other hand, analysis of sessions of learners who use Learn Your Way can provide extensive information, and we plan to pursue this in the future. For example, we recorded which Learn Your Way components were used by the different subjects. One interesting observation was that the majority of participants in the study used at least one content transformation

| Participants were asked:                                                                                                                             | Educational Tool Condition |                     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | ------------------- |
| To what extent do you agree or disagree with the following statements? % somewhat agree or strongly agree                                            | Learn Your Way n=30        | Digital Reader n=30 |
| I felt like today's educational tool made me more comfortable taking the assessment.                                                                 | 100%                       | 70%                 |
| I felt like today's educational tool helped me gain a good understanding of the content.                                                             | 97%                        | 87%                 |
| I would like to use today's educational tool to support my learning needs in the future.                                                             | 93%                        | 67%                 |
| I would recommend this educational tool to other students to support their learning needs.                                                           | 90%                        | 80%                 |
| I found today's educational tool enjoyable to use.                                                                                                   | 90%                        | 57%                 |
| I felt like I performed well on the assessment.                                                                                                      | 87%                        | 67%                 |
| The educational tool I used today would make me more effective at learning compared to other educational tools I currently use at home or in school. | 83%                        | 40%                 |

Figure 10 | Learner responses to a survey given after the assessment. For each statement, shown is the percentage of students who agreed or strongly agreed with it for each cohort. Red down arrow indicates that Learn Your Way had significantly higher ratings than the Digital Reader (p < .05).

in addition to the immersive text, and all made use of quizzes during the learning session.

This work is the tip of the iceberg of the potential impact of generative AI on learning at large and learning personalization in particular. The Learn Your Way experience could be extended in many ways, and the element of personalization could be explored and pushed much further. Additional learner attributes could be researched, through both explicit and implicit signals. The system could be made more adaptive, by dynamically adjusting the learning material to the performance of the learner on assessment components (e.g., to focus on learning gaps), or to the individual needs and difficulties of the learners. More interactive elements could be added to increase student interactions, both to obtain more signals from the learner and to create a more effective learning experience. Learn Your Way could be embedded in learning platforms, in ways that will provide teachers with control and insights into the learning process. The above feedback processes could then be designed to optimize learning efficacy.

Above all, Learn Your Way demonstrates how the imaginative use of generative AI, grounded in solid learning science principles and crafted and evaluated with pedagogical experts, opens up exciting opportunities to enhance learning.

### References

\[1] Matthew L Bernacki, Meghan J Greene, and Nikki G Lobczowski. A systematic review of research on personalized learning: Personalized by whom, to what, how, and for what purpose (s)? Educational Psychology Review, 33(4):1675-1715, 2021.

\[2] Atikah Shemshack and Jonathan Michael Spector. A systematic literature review of personalized learning terms. Smart learning environments, 7(1):33, 2020.

\[3] Shaaron Ainsworth. The functions of multiple representations. Computers and Education, 33(2): 131-152, 1999. ISSN 0360-1315. doi: [https://doi.org/10.1016/S0360-1315(99)00029-9](https://doi.org/10.1016/S0360-1315\(99\)00029-9). URL [https://www.sciencedirect.com/science/article/pii/S036013159900299](https://www.sciencedirect.com/science/article/pii/S036013159900299).

\[4] James M Clark and Allan Paivio. Dual coding theory and education. Educational psychology review, 3(3):149-210, 1991.

\[5] Olga Chernikova, Daniel Sommerhoff, Matthias Stadler, Doris Holzberger, Michael Nickl, Tina Seidel, Enkelejda Kasneci, Stefan Küchemann, Jochen Kuhn, Frank Fischer, and Nicole Heitzmann. Personalization through adaptivity or adaptability? a meta-analysis on simulationbased learning in higher education. Educational Research Review, 46:100662, 2025. ISSN 1747-938X. doi: [https://doi.org/10.1016/j.edurev.2024.100662](https://doi.org/10.1016/j.edurev.2024.100662). URL [https://www.sciencedirect.com/science/article/pii/S1747938X2400071X](https://www.sciencedirect.com/science/article/pii/S1747938X2400071X).

\[6] Vincent Aleven, Elizabeth A McLaughlin, R Amos Glenn, and Kenneth R Koedinger. Instruction based on adaptive learning technologies. Handbook of research on learning and instruction, 2: 522-560, 2016.

\[7] Jan L Plass and Shashank Pawar. Toward a taxonomy of adaptivity for learning. Journal of Research on Technology in Education, 52(3):275-300, 2020.

\[8] Barry J Zimmerman and Dale H Schunk. Self-regulated learning and performance: An introduction and an overview. Handbook of self-regulation of learning and performance, pages 15-26, 2011.

\[9] John Hattie. Visible learning: A synthesis of over 800 meta-analyses relating to achievement. routledge, 2008.

\[10] Ian Clark. Formative assessment: Assessment is for self-regulated learning. Educational psychology review, 24(2):205-249, 2012.

\[11] LearnLM Team, Abhinit Modi, Aditya Srikanth Veerubhotla, Aliya Rysbek, Andrea Huber, Ankit Anand, Avishkar Bhoopchand, Brett Wiltshire, Daniel Gillick, Daniel Kasenberg, Eleni Sgouritsa, Gal Elidan, Hengrui Liu, Holger Winnemoeller, Irina Jurenka, James Cohan, Jennifer She, Julia Wilkowski, Kaiz Alarakyia, Kevin R. McKee, Komal Singh, Lisa Wang, Markus Kunesch, Miruna Pislar, Niv Efron, Parsa Mahmoudieh, Pierre-Alexandre Kamienny, Sara Wiltberger, Shakir Mohamed, Shashank Agarwal, Shubham Milind Phal, Sun Jae Lee, Theofilos Strinopoulos, Wei-Jen Ko, Yael Gold-Zamir, Yael Haramaty, and Yannis Assael. Evaluating gemini in an arena for learning, 2025. URL [https://arxiv.org/abs/2505.24477](https://arxiv.org/abs/2505.24477).

\[12] J.P. Kincaid. Derivation of New Readability Formulas: (automated Readability Index, Fog Count and Flesch Reading Ease Formula) for Navy Enlisted Personnel. Research Branch report ; 8-75. Chief of Naval Technical Training, Naval Air Station Memphis, 1975. URL [https://books.google.co.il/books?id=\_Wq0zgEACAAJ](https://books.google.co.il/books?id=_Wq0zgEACAAJ).

\[13] J Peter Kincaid, Robert P Fishburne Jr, Richard L Rogers, and Brad S Chissom. Derivation of new readability formulas (automated readability index, fog count and flesch reading ease formula) for navy enlisted personnel. Technical report, NAVAL TECHNICAL TRAINING COMMAND MILLINGTON TN RESEARCH BRANCH, 1975.

\[14] Courtney Hattan, Patricia A Alexander, and Sarah M Lupo. Leveraging what students know to make sense of texts: What the research says about prior knowledge activation. Review of Educational Research, 94(1):73-111, 2024.

\[15] Paul Black and Dylan Wiliam. Developing the theory of formative assessment. Educational Assessment, Evaluation and Accountability (formerly: Journal of personnel evaluation in education), 21(1):5-31, 2009.

\[16] L.M. Earl. Assessment as Learning: Using Classroom Assessment to Maximize Student Learning. Experts on Assessment Kit. SAGE Publications, 2013. ISBN 9781452242972. URL [https://books.google.co.il/books?id=XKhl7I01o6MC](https://books.google.co.il/books?id=XKhl7I01o6MC).

### Contributions and Acknowledgments

### Core Contributors.

The following individuals made core contributions to the work described in this report. This list is ordered alphabetically, and does not indicate ranking of contributions:

Alicia Martin, Amir Globerson, Amy Wang, Anirudh Shekhawat, Anisha Choudhury, Anna Iurchenko, Avinatan Hassidim, Ayca Çakmakli, Ayelet Shasha Evron, Charlie Yang, Courtney Heldreth, Diana Akrong, Gal Elidan, Hairong Mu, Ian Li, Ido Cohen, Katherine Chou, Komal Singh, Lev Borovoi, Lidan Hackmon, Lior Belinsky, Michael Fink, Niv Efron, Preeti Singh, Rena Levitt, Shashank Agarwal, Shay Sharon, Tracey Lee-Joe, Xiaohong Hao, Yael Gold-Zamir, Yael Haramaty, Yishay Mor, Yoav Bar Sinai, Yossi Matias

### Acknowledgments.

We completed this work as part of the LearnLM effort—a cross-Google project, with members from Google DeepMind, Google Research, Google LearnX, and more. This tech report represents only a small part of the wider effort, and only lists team members who made direct contributions to this report. Special thanks to Ben Gomes, Irina Jurenka, James Manyika, Julia Wilkowski and Muktha Ananda for invaluable feedback.

### A. Source Materials

The PDFs used as the souce-of-truth for the pedagogical evaluations were all thanks to OpenStax. The 10 varied PDFs used are listed in the table below:

| Category      | Title                                                      |
| ------------- | ---------------------------------------------------------- |
| World History | Early Human Evolution and Migration                        |
| World History | The Ancient Roman Economy                                  |
| World History | The Cold War Begins                                        |
| Biology       | Evolution of Seed Plants                                   |
| Biology       | Disruptions in the Immune System                           |
| Physics       | Newton's Third Law of Motion                               |
| Economy       | How To Organize Economies: An Overview of Economic Systems |
| Astronomy     | Motions of Satellites and Spacecraft                       |
| Sociology     | Theories of Self-Development                               |
| Psychology    | Sleep and Why We Sleep                                     |

### B. Rubrics for Pedagogical Evaluations

|                                                   |                                                                                                                                                                | AGREE(1)                                                                                                                                                                                                                                                                                                                                                                                               | NEUTRAL/PARTIAL(0.5)                                                                                                                                                                                                                                                                                                           | DISAGREE(0)                                                                                                                                                                                                                                                                                  |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Accuracy                                          | Faithful to the source and accurate                                                                                                                            | The generated content is consistent with the source resource. Does not misrepresent or misconnect concepts. Captures and clearly conveys the source's main arguments, supporting key claims. The organization of the content(when applicable) is accurate and factual.                                                                                                                                 | Content does not add or alter main concepts, but might have minor inaccuracies, or miss concepts, relationships, or arguments.                                                                                                                                                                                                 | Content contains claims or arguments that counter the source, major inaccuracies, or misses major concepts.                                                                                                                                                                                  |
| Coverage                                          | Completeness of representation of source messages                                                                                                              | The generated content covers the source content, faithfully preserving the links and structure of the original. The source hierarchy is preserved. All key/top level concepts and LOs are covered, but immaterial details may be omitted to support learnability. For global constructs - global coverage. For local constructs - local coverage,i.e. projected to the scope covered by the construct. | Content mostly covers the structure and subject matter of the source, but is missing some key concepts, relationships, or messages. Source hierarchy is not preserved, some immaterial details are carried over while more important elements omitted.                                                                         | Key subject matter or structure is missing from the generated content.                                                                                                                                                                                                                       |
| Emphasis                                          | Precisely prioritizes core concepts. Identifies the source hierarchy, and optimal chooses and places key messages.                                             | The generated content focuses on the key concepts/relationships in the source, not trivial/marginal/esoteric concepts. For global constructs:The hierarchy of elements correlates with the knowledge/skills hierarchy of the source. For local constructs:The generated content focuses on the prominent elements in the source.                                                                       | Choice of elements partially reflects source hierarchy or intentions by including most of the core concepts, but might miss an important concept or emphasis some trivial material along with the core concepts.                                                                                                               | Choice of elements appears arbitrary, and does not align with source hierarchy or intents.Generated content highlights immaterial concepts or relationships instead of key messages.                                                                                                         |
| Engagement                                        | Positive, pleasant and purposeful user experience                                                                                                              | The generated content evokes positive emotional response.The tone,aesthetics,narrative,playfulness,and elements of personalization create a fun and captivating learning experience.                                                                                                                                                                                                                   | Content does not consistently engage or evoke a positive response from the user.Limited use of playfulness or personalization. Utility and relevance to user is present,but might not be clear to the user.                                                                                                                    | Content is dry,boring,does not attempt to evoke positive emotions.Monotonous single-track presentation.No perceived value to the user.                                                                                                                                                       |
| Cognitive load                                    | Effective use of cognitive effort                                                                                                                              | Content is well organized,clear and concise.Accessible language.Key points emphasized.Uses examples,analogies,narrative,and multiple representations effectively to enhance understanding.                                                                                                                                                                                                             | Content is fairly well structured but some sections could be improved.Phrasing is occasional difficult.Some unhelpful redundancy.Limited use of examples,analogies,narratives,or multiple representations to enhance learning.                                                                                                 | Content is poorly structured,and may included large overwhelming blocks of text.No clear hierarchy.The language level is inconsistent,incorporating difficult terms or oversimplifying for the users level.No use of aids such as examples,analogies,narratives,or multiple representations. |
| Active Learning                                   | Promotes active learning that goes beyond recall and comprehension.                                                                                            | Content raises questions and encourages user to engage with subject matter and learning objectives in multiple levels:recall,comprehension,analysis,application,evaluation,and creation.                                                                                                                                                                                                               | Content mostly focuses on information delivery,and misses opportunities to engage user with learning objectives at multiple levels.                                                                                                                                                                                            | Content is purely informational and does not encourage higher order thinking.                                                                                                                                                                                                                |
| Deepen Metacognition                              | Promotes active self-reflection,monitoring,regulation,and improvement of thinking and learning processes.                                                      | Provides relevant and actionable feedback.Prompts active reflection and inspection of learning processes.Supports the user to identify,execute and monitor their learning plan.                                                                                                                                                                                                                        | Provides somewhat useful feedback,but may not be relevant or actionable.Prompts shallow reflection without consideration of learning processes,or limited consideration of learning plans.                                                                                                                                     | Flat informative content,no useful feedback,no prompts for reflection or managing and regulating learning.                                                                                                                                                                                   |
| Motivation & curiosity                            | Encourages users to persist and deeply engage with learning processes.Incentivizes cognitive effort.                                                           | Uses a consistently supportive tone,encouraging user to persist.Maintains an optimal level of challenge,between too easy(boring)and too hard(frustrating).Highlights the relevance of the subject matter to the user,their concerns and interests.Empowers user autonomy by clearly marking available choices of learning depth,pace,order and focus.                                                  | Tone is occasional unsupportive,or encouragement is sporadic.Challenges are not consistently aligned to an optimal challenge level for the user.Limited use of examples and real-life connections to highlight subject matter relevance to user's concerns and interests.User choices in the learning process are not evident. | Tone is not supportive or encouraging.Challenges are missing,or at inappropriate level for the user.No evident links from the subject matter to user concerns and interests.No choice in learning path.                                                                                      |
| Adaptability & Personalization                    | Learning experience is adjusted to fit the user interests,preferences and characteristics.                                                                     | Content is fully adjusted to user's interests,age group,grade level,language proficiency,level of focus and attention,motivation,and preferences.Challenges are appropriate for the user's capabilities and intents.                                                                                                                                                                                   | Content acknowledges user's preferences and characteristics,but is only partially adjusted.Challenges are not consistently aligned with user capability or intent.                                                                                                                                                             | Content ignores user preferences and characteristics,"one size fits all".                                                                                                                                                                                                                    |
| Clarity of Learning Intentions & Success Criteria | Content clearly articulates what is being learned(learning intentions/goals/objectives)and how users will know if they have been successful(success criteria). | Learning objectives are clearly stated and user-friendly.Specific rubrics or benchmarks define what successful completion or understanding looks like.Content directly aligns with stated LOs and criteria(if provided).                                                                                                                                                                               | Learning objectives stated but not elaborated or demonstrated.Success criteria are vague or confusing for the user.Content alignment with LOs partial or unintuitive(if provided).                                                                                                                                             | Learning objectives not specified,unclear or inconsistent.No evident success criteria.Content misaligned with LOs(if provided).                                                                                                                                                              |

Table 1 | Pedagogical rubrics used by experts to rate the various components of Learn Your Way. Raters were also given the option of marking N/A or can't assess for each dimension.
