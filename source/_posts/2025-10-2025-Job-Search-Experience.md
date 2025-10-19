title: My 2025 Job Search Experience
date: 2025-10-17 17:04:52
tags:
---

Here I’d like to share some experience (mostly lessons) searching for a job in the tech industry over the last couple of months. It’s no longer secret that in 2025, the tech job market is pretty tough. I also hope my experience can be somewhat useful to you.
<!--more-->

# Recap

Earlier this year I got laid off from a mid-size startup. The company wasn't doing well financially. I didn't see it coming. I received the news early in the morning through a mandatory Zoom call.

My first reaction was actually a sense of relief. I could finally take my minds off work for a while. All the bugs, remaining tickets, milestones, and scheduled meetings suddenly all went away.

I went home and stayed with my family for about a month. It was during this completely off-time with old friends, family, and relatives.

After the break I went back to my apartment in the Bay Area for interviews, which went on for a couple of months. I instinctively rushed into applications and interviews and suffered some immediately setbacks. This brought me through some emotional roller-coasters and several more rounds of trying, before finally landing on offers.
It took me some time to realize how brutal the job market was, and how unprepared I was. At first I felt my confidence dinged pretty hard. And I took some more time for adjustments after each round.

I interviewed at both startups and big companies. I moved to second round with more famous names like Meta, OpenAI, Applied Intuition, Intuit, but all with no luck. I interviewed several other mid-size startups, and got in touch with a bunch of series-A and seed round startups, too. In total I interviewed at ~20 companies in total, ended up with two offers after almost 5 months of effort.

During this time I also got approached by some of my friends who were working on a pre-seed stealth startup on AI agents. I helped them build the basic backend infra in Python while I was looking for a new job.
This prolonged the process since I needed to redirect some of my attention to working instead of preparing for a job but it was still pretty worth it.

In the end I decided to move on to the job offer I received, which is a more established AI infra company. I think overall it fits my personal interests more.
But I did learn a lot in this process too. That's for another time and another story.

# Preparing

If I had to do it all over again, I'll definitely come to interviews more prepared instead of rushing into interviews, especially in this job market. I think I got lucky at interviews before. But this job market will definitely scrutinize you harder with tougher problems. I missed a few good opportunities since I performed pretty badly at the start.
I made some really dumb mistakes, and got stuck at really easy problems.

I was shooting for a Senior Software Engineer role. And my interest was on backend development, AI systems and infrastructure. So my preparation was focused on that, starting from basic coding exercises, to backend specific knowledge.

## Resume and Previous Experience

I didn't realize how unprepared I was until I discussed my resume and interview experience with one of my friends, who provided some valuable advises on presenting myself and my work. This is actually a rather important step that I missed.

The resume needed to stand out from other candidates. It need to provide detailed explanation of my contributions, all the rationale behind my technical decisions, and show that I could solve complex technical problems along with the team. This is the first step of providing a good impression for the interviewers and the hiring manager.

After discussing with my friend, I revised my resume and my self-introduction at interviews.

## Programming exercises

LeetCode, however you like it, it's still the go-to source for testing programming at interviews.

I wasn't particularly good at it. After a few failed interviews with easy problems, I spent a little more time going through some medium-level questions of each category. It turns out that most of the interviews I experienced were testing at roughly medium-level (except for Amazon and Bytedance, which was insanely hard).

## Machine Learning Basics

I also prepared myself with the basics of Machine Learning knowledge. There are AI infra positions which
requires an understanding of at least the fundamentals of AI/ML, especially the inner mechanisms of LLMs.

I took a quick look at the _Hands-On Machine Learning with Scikit-Learn, Keras, and Tensorflow_, and ByteByteGo's _Machine Learning System Design Interviews_. Even though I wasn't going for a ML engineer role (which requires expertise in building AI models), I thought it was it was helpful for getting an AI engineer or AI infra role.

Also went through some latest blog posts and papers (like this literature review: https://arxiv.org/abs/2506.21901) on LLM inference.

## System Design

I recommend ByteByteGo's books on System Design, the very nominal DDIA book (_Design Data Intensive Applications_). There are a lot of good mock interviews on YouTube as well.

In retrospect I failed pretty bad at System Design interviews since I didn't get ready well. I estimate that a good majority of my interviews failed from this.

One thing I realized about System Design interviews is that, **watching other people do it is really different to actually work through problems yourself!** My mistake was preparing by watching a lot of YouTube videos and nodding along. And I learned it the hard way that this isn't sufficient to get ready. And I unfortunately realized this during interviews when I sweat like hell getting grilled by interviewers.

To succeed in System Design interviews, you need to think deeply about each tech decisions, all the trade-offs you make, and be ready to defend all of them. Devils are always in the details. And this doesn't come with at least a good understanding of each component (e.g. Redis, Kafka, Cassandra, ...) and some experience and/or exercise. Memorizing what you watched on YouTube doesn't close all the gaps.

Some of the system design questions can get pretty tricky. And I realized that I don't have all the experience building with these components. Later I thought that a better way to prepare for these questions might be to actually getting hands dirty building something small with them. But that's for another story.

## Computer Science 101

Not all companies test this, and I was not well prepared for it. But in retrospect I think it's nice to also polish your knowledge on the domain knowledge you're specialized in, from Computer Science basics (e.g. what is the difference between a process and thread), to more specialized knowledge (e.g. programming language internals: what's a goroutine in Golang? How to use async in Python?)

Some companies do test you with this, especially you're targeting system, infra, or backend roles. Same with other specialties too, whether you're frontend, mobile, ML, or SRE, it's good to refresh on the fundamentals.

# Networking and Applying

This one I'm particularly bad at. It was after a long time did I realized that I could have reached out to more people and asked for referral or even guidance. Communication and networking was never my strong suit.

I reached out to some of my friends and former colleagues. Mostly of my interviews still came from getting in touch with recruiters on LinkedIn. Applications from myself was around 1/20, and referrals was 1/5. Most of my interviews were received through recruiter contacts through LinkedIn and SimplyHired.

In retrospect, I guess building a brand on LinkedIn or other social media might also be a good strategy of getting noticed more in times of need.

# Interviewing

## Formats

Most of the big companies and more established startups provided pretty standard interviewing formats:

- Basic phone screen/online screen + LeetCode.
- A round of (virtual) onsite interview of coding + system design.
- Potentially another round of coding + system design.
- Behavior interviews, usually with team lead or hiring manager.
- If lucky, final offer.

Or for some startups:

- Take home exercise in limited time, use as much AI as you want.
- Onsite chat with founders.

Coding rounds are becoming more interesting. I cannot disclose too many details, but I noticed a few interesting trends:

- Traditional companies use LeetCode more, such as Nvidia, Amazon, TikTok, Meta, etc. They follow the abovementioned procedure pretty closely.
- Startups are somehow getting much more creative with testing coding skills. From my experience they might ask you to:
	- Implement an actual small system or component, derived from a real system, to show your coding skills. And write test cases to verify your solution. It often comes with levels of updated requirements and levels of complexity.
	- Dig deep into a specific area (Python thread, ML knowledge, PyTorch knowledge, debugging thought process), e.g. to design a small system with multiple threads and handle synchronization.
	- Emulate code review. A co-worker submitted a GitHub PR. Now review and actively comment on the PR with the interviewer.
	- Emulate debugging, e.g. when an app is "slow", how do you find the root-cause? I actually enjoyed conversations like these. It was like a technical discussion and deep-dive.

The shift away from LeetCode is quite refreshing. I know many people in the industry dislike LeetCode. And some new style of questions mentioned above really test your experience and deep knowledge about something.

With AI being prevalent, I strongly believe there needs to be a shift in the industry on interviews, too. I don't have a definitive answer on it yet but I have no doubt we need to shift towards more innovative ways to examine a programmer's skills.

## Startups vs Big Companies

I also interviewed at some early stage startups. And they're drastically different experiences. Each startup is unique, heavily influenced by the founders' or founding engineers' personal touch.

Some founders particularly value communication skills. And some founders value your sense of product and acuteness for the market too (which I'm pretty bad at). I actually got turned down once because the founder thought I don't have the communication skills that their team needed.

What they're looking for is also way more different than regular engineers who punch cards at big companies. It's not just 9-6-6, but requires a strong sense of ownership, incredible passion, and a sense of mission. Skill wise, aside from being able to build from scratch, they require a lot of soft skills like adaptability, communication, and good product sense. Interviewing at startups can feel more unstructured, has more unexpected elements, but it's definitely more demanding. Also networking is more important, since startups have less resource for recruiting and building trust.

I appreciate my experience talking to some of the founding teams. It gave me a peek of their enthusiasm and the way they think about problems. In a few years I might come to interviewing startups again and this experience can help.

## The AI Engineering Landscape

AI Engineering is a hot topic right now. So many people are talking about it, but there's no clear definition of what it really means. It's an umbrella concept that covers a number of smaller specialties. Based on my experience interacting with various companies, I've identified the following key areas of focus.

- Lower level system: design and write high-performing custom CUDA kernels, tuning compilers, optimize computation on GPUs.
- AI infrastructure: there are startups that requires building infra, like cloud native, micro-service applications. Building AI inference serving or training infrastructure.
- Machine Learning Engineer: building data pipelines for ML-specific. This is slightly different, which requires knowledge and experience in big data processing.
- Machine Learning Expert: The core of algorithm development. This area involves designing, building, and refining the machine learning models that power AI features.
- AI agents: this is a really hot and popular topic. There are a bunch of AI agents startups.

These are all exciting opportunities. And some companies requires a mix of these skills.

# Mental Health

Interviewing in this job market is no longer a sprint. Treat it like a marathon and keep adjusting your mind for the upcoming challenges. It can be quite onerous to go through a full round of interview, getting examined hard, with these efforts and finally getting a rejection in return. It can be particularly mentally taxing if you have to go through multiple interviews at the same time and multiple rejections.

If necessary, get some downtime. I later realized It's completely ok to treat your mental health instead of powering through. Your energy is like a battery, and it's wise to recharge it when it's low. Take this opportunity to take a break. Or travel a little if that helps.

Find ways to connect with your friend, or a professional for help. During this process I spent most of my time on my own trying to push through it, and I felt really tiresome for many times. Later I tried to completely break free from all the work, preparation, or even thinking about the interviews.

# Lessons Learned

To summarize, my lessons for interviews boil down to:

- Prepare your resume and your self-introduction with more thought.
- Try to showcase your thought process, the challenges you solved, the impact you made. This will help in the behavioral interviews, too.
- Spend more time to get prepared. Polish the basics coding skills, CS knowledge, and knowledge for the specialization.
- Network. This is the time to use your networking in time of stress. And sometimes your friends can help you more than you could expect. Many great opportunities come from referrals.
- If you're interviewing at a startup, understand the startup you're applying for well enough. Do a thorough research. Even try their product a little. Big companies look for employees, startups look for co-founders. Show your passion and care really make a difference.
- Treat yourself well. If you're unfortunately going through a lay-off, it's wise to take care of your own mental health to sail through this hardship.

# For Interviewers

At last, my two cents for all the companies hiring people, too. Some companies really value their whole interview experience, some felt more outright chaotic. I had an experience where the interviewers showed up, looking distracted, and felt even more unprepared than I was. We went through some easy problems, and I got a rejection letter a week later with no explanations. Some companies simply ghost you for weeks.

On the other hand I had some really good experience, too, like OpenAI and Intuit, even though I didn't land on an offer with them. The recruiters even forwarded some feedback from the team after the interviews. And this feedback helped me quickly re-focus on my weak points, since sometimes I made mistakes on places I didn't even notice.

Also I know it's often difficult to provide any feedback to candidates. Many companies avoid doing for fear of complications. But I personally was really thankful when they did.

In short: treat your candidates well, instead of like treating another expendable. A little bit of an effort in this can go a long way. I'll prioritize a company with a healthy working environment than the ones who shows toxicity during interview processes.

# Closing Thoughts

I had a gap of a couple of months, but all is not lost. I had some good experience working part time at a startup and learned a lot through getting contact with some great companies in the industry. I gained a good knowledge of what I lack, and what I need.

The industry is currently evolving at an unprecedented pace, with a dramatic refocus on AI. I'm confident that AI has the transformative potential, and I'm excited to contribute my share of effort. But like fast waters, changes also mean risks. I'll need to keep an open eye on the shifts and turns to navigate through.

I feel lucky that I finally landed on my current offer. From now on I'll focus on the new work. And I believe with good work, good luck follows.
