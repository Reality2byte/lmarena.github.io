---
layout: distill
title: "Finally. Search Arena."
description: 
giscus_comments: true
date: 2025-04-09
featured: true
thumbnail: assets/img/blog/explorer/explorer.png
authors:
  - name: Mihran Miroyan*
    url: "https://mmiroyan.github.io"
    affiliations:
      name: UC Berkeley
  - name: Tsung-Han Wu*
    url: "https://tsunghan-wu.github.io"
  - name: Logan King
    url: "https://www.linkedin.com/in/logan-king-8a4267281"
  - name: Tianle Li
    url: "https://codingwithtim.github.io"
  - name: Anastasios N. Angelopoulos†
    url: "http://angelopoulos.ai"
  - name: Wei-Lin Chiang†
    url: "https://infwinston.github.io"
  - name: Joseph E. Gonzalez†
    url: "https://people.eecs.berkeley.edu/~jegonzal"
---

## TL;DR

1. We introduce **Search Arena**, a crowdsourced in-the-wild evaluation platform for search-augmented LLM systems based on human preference. Unlike LM-Arena or SimpleQA, our data focuses on current events and diverse real-world use cases (see [Sec. 1](#why-search-arena)).
2. Based on 5k human votes (03/18–04/06), the leaderboard right below ranks Gemini-2.5-Pro-Grounding at the top, followed by Perplexity's Sonar models, Gemini-2.0-Flash-Grounding, and OpenAI's GPTs. Standardizing citation styles had minimal effect on rankings (see [Sec. 2](#leaderboard)).
3. Three features show strong positive correlation with human preference: response length, citation count, and specific web sources like YouTube and GitHub (see [Sec. 3](#analyses)).
4. We open-sourced the [🤗 5k-human-preference-dataset](https://huggingface.co/datasets/lmarena-ai/search-arena-v1-5k) and a [⚙️ Colab notebook](https://colab.research.google.com/drive/1Vb92s3O9RZ0aOs5c953SzaB2HniVcX4_) for leaderboard analysis. Try the [🌐 Search Arena](https://lmarena.ai/?search) and see [Sec. 4](#futurework) for what's next.

<div id="fig1">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/main_bootstrap_elo_rating.html' }}" frameborder='0' scrolling='no' height="400px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 1. Search Arena leaderboard.</p>

<h2 id="why-search-arena">1. Why Search Arena?</h2>

Web search is undergoing a major transformation. Search-augmented LLM systems integrate dynamic real-time web data with the reasoning, problem-solving, and question-answering capabilities of LLMs. These systems go beyond traditional retrieval, enabling richer human–web interaction. The rise of models like Perplexity’s Sonar, OpenAI's GPT-Search, and Google's Gemini-Grounding highlights their growing impact.

But how should they be evaluated? Static benchmarks like SimpleQA focus on factual accuracy on challenging questions, but that’s only one piece. These systems are used for diverse tasks—coding, research, recommendations—so evaluations must also consider how they retrieve, process, and present information. Understanding this requires studying real-world use and human preferences.

To this end, we developed search arena, aiming to (1) enable crowd-sourced evaluation of search-augmented LLMs and (2) release a diverse, in-the-wild dataset of user–system interactions.

Since our [initial launch](https://x.com/lmarena_ai/status/1902036561119899983) on March 18th, we've collected over 8,500 votes across 10+ models. In short, we used the filtered [🤗 5k-human-preference-dataset](https://huggingface.co/datasets/lmarena-ai/search-arena-v1-5k) to calculate the leaderboard with this [⚙️ Colab notebook](https://colab.research.google.com/drive/1Vb92s3O9RZ0aOs5c953SzaB2HniVcX4_). Below, we detail the data and models involved.

<h3>A. Data</h3>

<b>Data Filtering and Citation Style Control.</b> We found that each model provider uses a unique *inline citation style*. While these styles can compromise anonymity, certain presentation formats may also be more user-friendly or preferred. To balance fairness and enable study on this effect, we introduced <em>"style randomization"</em>: responses are shown either in a standardized format or in the original format (i.e., the citation style agreed upon with the model provider). 
<details>
  <summary>Click to view citation style comparisons.</summary>
  <div style="margin-top: 1rem;">
    <img 
      src="/assets/img/blog/search_arena/04092025/gemini_formatting_example.png" 
      alt="Google's Gemini citation formatting comparison"
      style="width: 100%; max-width: 1000px; border: 1px solid #ccc; border-radius: 8px;"
    />
  </div>
  <p>(1) Google's Gemini Formatting: standardized (left), original (right)</p>
  <div style="margin-top: 1rem;">
    <img 
      src="/assets/img/blog/search_arena/04092025/ppl_formatting_example.png" 
      alt="Perplexity's Sonar citation formatting comparison"
      style="width: 100%; max-width: 1000px; border: 1px solid #ccc; border-radius: 8px;"
    />
  </div>
  <p>(2) Perplexity’s Sonar Formatting: standardized (left), original (right)</p>
  <div style="margin-top: 1rem;">
    <img 
      src="/assets/img/blog/search_arena/04092025/gpt_formatting_example.png" 
      alt="OpenAI's GPT citation formatting comparison"
      style="width: 100%; max-width: 1000px; border: 1px solid #ccc; border-radius: 8px;"
    />
  </div>
  <p>(3) OpenAI's GPT Formatting: standardized (left), original (right)</p>
</details>

This approach mitigates de-anonymization while allowing us to analyze how citation style impacts user judgment (see the citation analyses subsection [here](#citation_analyses)). After updating and standardizing citation styles in collaboration with providers, we filtered the dataset to include only post-update battles—resulting in ~5,000 clean samples for leaderboard computation and further analysis. 

<b>Comparison to Existing Benchmarks.</b> To highlight what makes Search Arena unique, we compare our collected data to [LM-Arena](https://arxiv.org/abs/2403.04132) and [SimpleQA](https://arxiv.org/abs/2411.04368). As shown in [Fig. 2](#fig2), Search Arena prompts focus more on current events, while LM-Arena emphasizes coding/writing, and SimpleQA targets narrow factual questions (e.g., dates, names, specific domains). [Tab. 1](#tab1) shows that Search Arena features longer prompts, longer responses, more turns, and more languages compared to SimpleQA—closer to natural user interactions seen in LM-Arena.

<div id="fig2">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/topical_distribution_plot.html' }}" frameborder='0' scrolling='no' height="450px" width="100%"></iframe>
</div>
<p style="color:gray; text-align: center;">Figure 2. Top-5 topic distributions across Search Arena, LM Arena, and SimpleQA. We use 
  <a href="https://blog.lmarena.ai/blog/2025/arena-explorer/">Arena Explorer (Tang et al., 2025)</a> to extract topic clusters from the three datasets.</p>


<table id="tab1" style="margin: 0 auto; border-collapse: collapse; text-align: center;">
  <thead>
    <tr>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;"></th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">Search Arena</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">LM Arena</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">SimpleQA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 8px;">Languages</td>
      <td style="padding: 8px;">10+ (EN, RU, CN, …)</td>
      <td style="padding: 8px;">10+ (EN, RU, CN, …)</td>
      <td style="padding: 8px;">English Only</td>
    </tr>
    <tr>
      <td style="padding: 8px;">Avg. Prompt Length (#words)</td>
      <td style="padding: 8px;">88.08</td>
      <td style="padding: 8px;">102.12</td>
      <td style="padding: 8px;">16.32</td>
    </tr>
    <tr>
      <td style="padding: 8px;">Avg. Response Length (#words)</td>
      <td style="padding: 8px;">344.10</td>
      <td style="padding: 8px;">290.87</td>
      <td style="padding: 8px;">2.24</td>
    </tr>
    <tr>
      <td style="padding: 8px;">Avg. #Conversation Turns</td>
      <td style="padding: 8px;">1.46</td>
      <td style="padding: 8px;">1.37</td>
      <td style="padding: 8px;">N/A</td>
    </tr>
  </tbody>
</table>

<p style="color:gray; text-align: center;">
Table 1. Prompt language distribution, average prompt length, average response length, and average number of turns in Search Arena, LM Arena, and SimpleQA datasets.</p>

<h3>B. Models</h3>

Search Arena currently supports over ten models from three providers: Perplexity, Gemini, and OpenAI. Unless specified otherwise, we treat the same model with different citation styles (original vs. standardized) as a single model. [Fig. 3](#fig3) shows the number of battles collected per model used in this iteration of the leaderboard.

By default, we use each provider’s standard API settings. For Perplexity and OpenAI, this includes setting the `search_context_size` parameter to `medium`, which controls how much web content is retrieved and passed to the model. In some cases, we explore specific features beyond the default: (1) For OpenAI, we test their geolocation feature in one model variant by passing a country code derived from the user’s IP address. (2) For Perplexity and OpenAI, we include variants with `search_context_size` set to `high`. Below is the list of models currently supported in Search Arena:

<table style="width:100%; border-collapse: collapse; text-align: left;">
  <thead>
    <tr>
      <th style="border-bottom: 1px solid #ccc; padding: 8px;">Provider</th>
      <th style="border-bottom: 1px solid #ccc; padding: 8px;">Model</th>
      <th style="border-bottom: 1px solid #ccc; padding: 8px;">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="5" style="padding: 8px;">Perplexity</td>
      <td style="padding: 8px;"><code>ppl-sonar</code></td>
      <td style="padding: 8px;">Default config</td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>ppl-sonar-pro</code></td>
      <td style="padding: 8px;">Default config</td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>ppl-sonar-pro-high</code></td>
      <td style="padding: 8px;"><code>search_context_size</code> set to <code>high</code></td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>ppl-sonar-reasoning</code></td>
      <td style="padding: 8px;">Default config</td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>ppl-sonar-reasoning-pro-high</code></td>
      <td style="padding: 8px;">Recently added, results coming soon</td>
    </tr>

    <tr>
      <td rowspan="2" style="padding: 8px;">Gemini</td>
      <td style="padding: 8px;"><code>gemini-2.5-pro-exp-03-25-grounding</code></td>
      <td style="padding: 8px;">With <code>Google Search</code> tool; originally displayed as <code>gemini-2.0-pro-exp-02-05-grounding</code> due to API routing<sup>†</sup></td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>gemini-2.0-flash-grounding</code></td>
      <td style="padding: 8px;">With <code>Google Search</code> tool</td>
    </tr>

    <tr>
      <td rowspan="4" style="padding: 8px;">OpenAI</td>
      <td style="padding: 8px;"><code>gpt-4o-mini-search-preview</code></td>
      <td style="padding: 8px;">Default config</td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>gpt-4o-search-preview</code></td>
      <td style="padding: 8px;">Default config</td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>gpt-4o-search-preview-high</code></td>
      <td style="padding: 8px;"><code>search_context_size</code> set to <code>high</code></td>
    </tr>
    <tr>
      <td style="padding: 8px;"><code>gpt-4o-search-preview-high-loc</code></td>
      <td style="padding: 8px;"><code>user_location</code> feature enabled</td>
    </tr>
  </tbody>
</table>

<p style="color:gray; font-size: 14px; text-align: center; margin-top: 8px;">
  Table 2. Models currently supported in Search Arena.
</p>

<p style="color:gray; font-size: 13px; max-width: 700px; margin: 0 auto;">
  <sup>†</sup> Until April 8, the model was displayed as <code>gemini-2.0-pro-exp-02-05-grounding</code> due to Google’s API routing,
  which internally mapped calls to <code>gemini-2.0-pro-exp-02-05</code> onto <code>gemini-2.5-pro-exp-03-25</code>.
  We've corrected the display. Also, we detected this behavior and only include battles where this routing rule was in effect for our analyses and leaderboard.
</p>

<div id="fig3">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/main_battle_count.html' }}" frameborder='0' scrolling='no' height="400px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 3. Battle count across ten models. The distribution is not even as we (1) added models after the first release and (2) filtered votes (described above).</p>

<h2 id="leaderboard">2. Leaderboard</h2>

We begin by analyzing pairwise win rates—i.e., the proportion of wins of model A over model B in head-to-head battles. This provides a direct view of model performance differences without relying on score aggregation. The results are shown in [Fig. 4](#fig4), along with the following observations:

- `gemini-2.5-pro-grounding` consistently surpasses all other models by a large margin.
- Among Perplexity models, <code>ppl-sonar-reasoning</code> outperforms the other three. There's no clear difference between <code>ppl-sonar-pro</code> and <code>ppl-sonar-pro-high</code> (~50% win rate), and even <code>ppl-sonar</code> beats <code>ppl-sonar-pro-high</code> (57% win rate). This suggests that increasing search context does not necessasary improve performance.
- Within OpenAI's models, larger search context improves performance—<code>gpt-4o-search-high</code> and <code>gpt-4o-search-high-loc</code> both outperform <code>gpt-4o-search</code>, and location-aware variants show a slight advantage.

<div id="fig4">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/main_pairwise_average_win_rate.html' }}" frameborder='0' scrolling='no' height="600px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 4. Pairwise win rates (Model A wins Model B), <code>tie</code> and <code>tie (bothbad)</code> votes.</p>

Now we build the leaderboard! Consistent with [LM Arena](https://lmarena.ai/), we apply the Bradley-Terry (BT) model to compute model scores. The resulting BT coefficients are then translated to Elo scale, with the final model scores and rankings displayed in [Fig. 1](#fig1) and [Tab. 3](#tab3). The confidence intervals are still wide, which means the leaderboard hasn’t fully settled and there’s still some uncertainty. But clear performance trends are already starting to show. Consistent with the pairwise win rate analysis in the previous section, `gemini-2.5-pro-grounding` tops the leaderboard by a substantial margin. It is followed by models from the `ppl-sonar` family, with `ppl-sonar-reasoning` leading the group. Then comes `gemini-2.0-flash-grounding`, and finally OpenAI models, with high search context models performing better.



<table id="tab2"  style="width: 100%; border-collapse: collapse; text-align: center;">
  <thead>
    <tr>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">Rank</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">Model</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">Arena Score</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">95% CI</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">Votes</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">Organization</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>1</td><td><code>gemini-2.5-pro-grounding</code></td><td>1138</td><td>+21/-20</td><td>685</td><td>Google</td></tr>
    <tr><td>2</td><td><code>ppl-sonar-reasoning</code></td><td>1100</td><td>+15/-15</td><td>1,268</td><td>Perplexity</td></tr>
    <tr><td>2</td><td><code>ppl-sonar-pro-high</code></td><td>1071</td><td>+16/-16</td><td>970</td><td>Perplexity</td></tr>
    <tr><td>2</td><td><code>ppl-sonar-pro</code></td><td>1071</td><td>+18/-15</td><td>1,084</td><td>Perplexity</td></tr>
    <tr><td>3</td><td><code>ppl-sonar</code></td><td>1068</td><td>+14/-14</td><td>1,087</td><td>Perplexity</td></tr>
    <tr><td>5</td><td><code>gemini-2.0-flash-grounding</code></td><td>1036</td><td>+19/-15</td><td>801</td><td>Google</td></tr>
    <tr><td>6</td><td><code>gpt-4o-search-high</code></td><td>1009</td><td>+16/-12</td><td>1,293</td><td>OpenAI</td></tr>
    <tr><td>7</td><td><code>gpt-4o-search-high-loc</code></td><td>1000</td><td>+20/-18</td><td>690</td><td>OpenAI</td></tr>
    <tr><td>7</td><td><code>gpt-4o-search</code></td><td>1000</td><td>+13/-15</td><td>1,073</td><td>OpenAI</td></tr>
    <tr><td>8</td><td><code>gpt-4o-mini-search</code></td><td>971</td><td>+18/-17</td><td>1,049</td><td>OpenAI</td></tr>
  </tbody>
</table>
<p style="color:gray; text-align: center;">
  Table 3. Search Arena leaderboard.
</p>

<h3 id="citation_analyses">Citation Style Analysis</h3>

Having calculated the main leaderboard, we can now analyze the effect of citation style on user votes and model rankings. For each battle, we record model A’s and B’s citation style — original (agreed upon with the providers) vs standardized.

First, following the the method in [(Li et al., 2024)](https://blog.lmarena.ai/blog/2024/style-control/), we apply style control and use the citation style indicator variable (1 if standardized, 0 otherwise) as an additional feature in the BT model. The resulting model scores and rankings do not change significantly from [the main leaderboard](#fig1). Although the leaderboard does not change, the corresponding coefficient is positive (0.068) and statistically significant (p<0.05), implying that standardization of citation style has a positive impact on model score.

We further investigate the effect of citation style on model performance, by treating each combination of model and citation style as a distinct model (e.g., `gpt-4o-search` with original style will be different from `gpt-4o-search` with standardized citation style). [Fig. 5](#fig5) shows the change in the arena score between the two styles for each model. Overall, we observe a consistent improvement in score with standardized citations across all models except `gemini-2.0-flash`. However, the differences remain within the confidence intervals (CI), and we will continue collecting data to assess whether the trend converges toward statistical significance.

<div id="fig5">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/og_vs_st.html' }}" frameborder='0' scrolling='no' height="400px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 5. Change in arena score for original vs standardized citation style for each model.</p>

<h2 id="analyses">3. Three Secrets Behind a WIN</h2>

After reviewing the leaderboard—and showing that the citation style doesn’t impact results all that much—you might be wondering: *What actually drives a model's win rate?*

To answer this, we used the method in [(Zhong et al., 2022)](https://arxiv.org/abs/2201.12323), a method that automatically proposes and tests hypotheses to identify key differences between two groups of natural language texts—in this case, human-preferred and rejected model outputs. In our implementation, we asked the model to generate 25 hypotheses and evaluate them, leading to the discovery of *three distinguishing factors* with statistically significant p-values, shown in [Tab. 4](#tab4).


<table id="tab4" style="width: 100%; border-collapse: collapse; text-align: left;">
  <thead>
    <tr>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">Feature</th>
      <th style="padding: 8px; border-bottom: 1px solid #ccc;">p-value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 8px;">References to specific known entities or platforms</td>
      <td style="padding: 8px;">0.0000114</td>
    </tr>
    <tr>
      <td style="padding: 8px;">Frequent use of external citations and hyperlinks</td>
      <td style="padding: 8px;">0.01036</td>
    </tr>
    <tr>
      <td style="padding: 8px;">Longer, more in-depth answers</td>
      <td style="padding: 8px;">0.04761</td>
    </tr>
  </tbody>
</table>

<p style="color:gray; text-align: center;">
  Table 4. Candidate key factors between the winning and losing model outputs.
</p>


### Model Characteristics

Guided by these findings, we analyze how these features vary across models and model families. 

[Fig. 6 (left)](#fig6) shows the distribution of average response length across models. Gemini models are generally the most verbose—<code>gemini-2.5-pro-grounding</code>, in particular, produces responses nearly twice as long as most Sonar or OpenAI models. Within the Sonar and OpenAI families, response length is relatively consistent. [Fig. 6 (right)](#fig6) shows the average number of citations per response. Sonar models cite the most, with <code>ppl-sonar-pro-high</code> citing 2-4x more than Gemini models. OpenAI models cite the fewest sources (2-2.5) with little variation within the group.

<div id="fig6">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/lenght_cit_features.html' }}" frameborder='0' scrolling='no' height="400px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 6. Average response length (left) and number of citations (right) per model.</p>

Beyond quantity, we also examine the <em>source domains</em> that models cite. We categorize retrieved URLs into ten types: YouTube, News (U.S. and foreign), Community & Blogs (e.g., Reddit, Medium), Wikipedia, Tech & Coding (e.g., Stack Overflow, GitHub), Government & Education, Social Media, Maps, and Academic Journals. [Fig. 7](#fig7) shows the domain distribution across providers in two settings: (1) all conversations, and (2) a filtered subset focused on Trump-related prompts. The case study helps examine how models handle current events. Here're three interesting findings:
1. All models favor authoritative sources (e.g., Wikipedia, <code>.edu</code>, <code>.gov</code> domains).
2. OpenAI models heavily cite news sources—68% overall and 92% for Trump-related prompts—while rarely citing YouTube.
3. Gemini prefers community/blog content, whereas Perplexity frequently cites YouTube. Perplexity also strongly favors U.S. news sources over foreign ones (3x more often).

<div id="fig7">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/domain_citations.html' }}" frameborder='0' scrolling='no' height="400px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 7. Distribution of cited domain types across models. Use the dropdown to switch between all prompts and a filtered Trump-related subset.</p> 

### Control Experiments

After analyzing model characteristics such as response length, citation count, and citation sources, we revisited the Bradley-Terry model with these features as additional control variables [(Li et al., 2024)](https://blog.lmarena.ai/blog/2024/style-control/). Here's the result we got:

- **Response length**: Longer responses are favored (coef = 0.204, p < 0.05).
- **Citation count**: More citations correlate with higher preference (coef = 0.234, p < 0.05).
- **Citation source categories**: Only tech/coding platforms (e.g., GitHub) and YouTube show significant positive effects. Other categories are negligible shown in [Fig. 8](#fig8).
- **Joint controls**: When controlling for all features together, only **response length** and **citation count** remained statistically significant.

<div id="fig8" style="display: flex; justify-content: center;">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/domain_citations_style_control_bootstrap_style_coefs.html' }}" frameborder='0' scrolling='no' height="400px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 8. Estimates (with 95% CIs) of style coefficients.</p>

Finally, we used all previously described features to construct a controlled leaderboard. [Fig. 9](#fig9) compares the original and adjusted arena scores after controlling for response length, citation count, and source types. Interestingly, when using all these features as control variables, the top five models all show a reduction in score, while the remaining models are largely unaffected. This narrows the gap between <code>gemini-2.0-flash-grounding</code> and most Perplexity models. Under this controlled setting, the leaderboard consolidates into four distinct performance tiers, as shown in [Tab. 5](#tab5).

<div id="fig9">
  <iframe src="{{ '/assets/img/blog/search_arena/04092025/style_control.html' }}" frameborder='0' scrolling='no' height="400px" width="100%"></iframe>
</div>

<p style="color:gray; text-align: center;">Figure 9. Arena scores before and after a controlled setting.</p>



<table id="tab5">
  <thead>
    <tr>
      <th>Model</th>
      <th>Rank Diff (Length)</th>
      <th>Rank Diff (# Citations)</th>
      <th>Rank Diff (Domain Sources)</th>
      <th>Rank Diff (All)</th>
    </tr>
  </thead>
  <tbody>
    <tr><td><code>gemini-2.5-pro-grounding</code></td><td>1→1</td><td>1→1</td><td>1→1</td><td>1→1</td></tr>
    <tr><td><code>ppl-sonar-reasoning</code></td><td>2→1</td><td>2→2</td><td>2→2</td><td>2→2</td></tr>
    <tr><td><code>ppl-sonar-pro-high</code></td><td>2→2</td><td>2→3</td><td>2→2</td><td>2→2</td></tr>
    <tr><td><code>ppl-sonar-pro</code></td><td>2→2</td><td>2→3</td><td>2→2</td><td>2→2</td></tr>
    <tr><td><code>ppl-sonar</code></td><td>3→2</td><td>3→3</td><td>3→3</td><td>3→2</td></tr>
    <tr><td><code>gemini-2.0-flash-grounding</code></td><td>5→3</td><td>5→2</td><td>5→3</td><td>5→2</td></tr>
    <tr><td><code>gpt-4o-search-high</code></td><td>6→7</td><td>6→3</td><td>6→6</td><td>6→3</td></tr>
    <tr><td><code>gpt-4o-search-high-loc</code></td><td>7→6</td><td>7→3</td><td>7→6</td><td>7→3</td></tr>
    <tr><td><code>gpt-4o-search</code></td><td>7→7</td><td>7→3</td><td>7→6</td><td>7→3</td></tr>
    <tr><td><code>gpt-4o-mini-search</code></td><td>8→8</td><td>8→8</td><td>8→8</td><td>8→8</td></tr>
  </tbody>
</table>

<p style="color:gray; text-align: center;">
  Table 5. The leaderboard rank differs when controlling for different subsets of features.
</p>

<h2 id="futurework">4. Conclusion & What’s Next</h2>

As search-augmented LLMs become increasingly popular, **Search Arena** provides a real-time, in-the-wild evaluation platform driven by crowdsourced human feedback. Unlike static QA benchmarks, our dataset emphasizes current events and diverse real-world queries, offering a more realistic view of how users interact with these systems.

From over 5,000 human votes, we found that **Gemini-2.5-Pro-Grounding** leads overall. User preferences are correlated to **response length**, **number of citations**, and **citation sources**. Citation formatting, surprisingly, had minimal impact.


We’ve open-sourced the [🤗 5k-human-preference dataset](https://huggingface.co/datasets/lmarena-ai/search-arena-v1-5k) and a [⚙️ Colab notebook](https://colab.research.google.com/drive/1Vb92s3O9RZ0aOs5c953SzaB2HniVcX4_). Try the [🌐 Search Arena](https://lmarena.ai/?search) now and see what’s next: 

- **Open participation**: We’re inviting model submissions from researchers and industry, and encouraging public voting.
- **Cross-task evaluation**: How well do search models handle general questions? Can LLMs manage search-intensive tasks?
- **Raise the bar for open models:** Can simple wrappers with search engine/scraping + tools like ReAct and function calling make open models competitive?

## Citation

```bibtex
@misc{searcharena2025,
    title = {Finally. Search Arena.},
    url = {https://blog.lmarena.ai/blog/2025/search-arena/},
    author = {Mihran Miroyan*, Tsung-Han Wu*, Logan Kenneth King, Tianle Li, Anastasios N. Angelopoulos†, Wei-Lin Chiang†, Joseph E. Gonzalez†},
    month = {April},
    year = {2025}
}

@inproceedings{chiang2024chatbot,
  title={Chatbot arena: An open platform for evaluating llms by human preference},
  author={Chiang, Wei-Lin and Zheng, Lianmin and Sheng, Ying and Angelopoulos, Anastasios Nikolas and Li, Tianle and Li, Dacheng and Zhu, Banghua and Zhang, Hao and Jordan, Michael and Gonzalez, Joseph E and others},
  booktitle={Forty-first International Conference on Machine Learning},
  year={2024}
}

@misc{stylearena2024,
    title = {Does Style Matter? Disentangling style and substance in Chatbot Arena},
    url = {https://blog.lmarena.ai/blog/2024/style-control/},
    author = {Tianle Li*, Anastasios Angelopoulos*, Wei-Lin Chiang*},
    month = {August},
    year = {2024}
}
```