---
layout: post
title: Evaluation and Solution of Urban Crawl 
tags: [lua文章]
categories: [topic]
---
这是我与同学组队在2017年2月参加美国数学建模大赛（Interdisciplinary Contest In Modeling
）的文章，比赛题目为：Sustainable Cities Needed

题目原文为：

Background:  
Many communities are implementing smart growth initiatives in an effort to
consider long range, sustainable planning goals. “Smart growth is about
helping every town and city become a more economically prosperous, socially
equitable, and environmentally sustainable place to live.”[2] Smart growth
focuses on building cities that embrace the E’s of sustainability—Economically
prosperous, socially Equitable, and Environmentally Sustainable. This task is
more important than ever because the world is rapidly urbanizing. It is
projected that by 2050, 66 percent of the world’s population will be
urban—this will result in a projected 2.5 billion people being added to the
urban population.[3] Consequently, urban planning has become increasingly
important and necessary to ensure that people have access to equitable and
sustainable homes, resources and jobs.  
Smart growth is an urban planning theory that originated in 1990’s as a means
to curb continued urban sprawl and reduce the loss of farmland surrounding
urban centers. The ten principles for smart growth are  
1 Mix land uses  
2 Take advantage of compact building design  
3 Create a range of housing opportunities and choices  
4 Create walkable neighborhoods  
5 Foster distinctive, attractive communities with a strong sense of place  
6 Preserve open space, farmland, natural beauty, and critical environmental
areas  
7 Strengthen and direct development towards existing communities  
8 Provide a variety of transportation choices  
9 Make development decisions predictable, fair, and cost effective  
10 Encourage community and stakeholder collaboration in development decisions  
These broad principles must be tailored to a community’s unique needs to be
effective. Thus, any measure of success must incorporate the demographics,
growth needs, and geographical conditions of a city as well as the goal to
adhere to the three E’s.

以下是我们小组的文章：

## Evaluation and Solution of Urban Crawl

### Abstract

In this paper, based on theproblems, we divide our thinking into 4 parts.

For task 1, we developa ’Successful Smart Growth’ index (SSG) to measure the
governance of urbancrawl based on the impact of the ten principles on the
three E’s and happiness.We used the rank correlation analysis and geometric
compare scale to considerhow the factors were affected by each principle.
Using this as a basis, wedetermine the influence level and furthermore the
weights of the three E’s andhappiness in the SSG index. Finally we calculated
the weighted sum of thenondimensionalized variance of three E’s and happiness
and used that tocalculate our SSG index.

For task 2, we appliedour model to two cities: TAKASAKI and Mobile City. Based
on our assumptions, weuse the financial condition of the annual statement and
the variance ratio atthe national level to represent the changes in three E’s
and happiness. Afterthat, we nondimensionalized the variance of three E’s and
happiness by usingthe algebraic model to calculate the variance ratio.
Finally, we gave score ofthe SSG index of TAKASAKI and Mobile City.

For task 3 & 4, weproposed six individual initiatives and developed a more
robust model — FSGPindex based on the situation of TAKASAKI and Mobile City.
According to theproblems in the cities, we proposed our smart growth plan
based on tenprinciples. Using the scale comparison method and the weighting
method, weobtained the SPG index. Then with the enlightenment of _the theory
of optimum population ( A. Sauvy )_ , we calculated theproper population
density for development by using the quadratic function andsymbolic-graphic
combination. Finally, we used profit loss equation to defineprofit loss
coefficient and optimized the SSG index by using the coefficient.

For task 5, using theconcept of task 4, we divided the result of population
increase into twoconditions. And we analyzed the changes of our initiatives in
two conditionsand prove the robustness of our model.

Synthesize tenprinciples, three E’s and happiness, we evaluated the success of
smart growthof cities and their plans.

​ **Key Words:** SSG index SmartGrowth FSGP index

### 1 Introduction

#### 1.1 Background

Urban development had come up to aseries of problems since 1970s. The boundary
of the city was getting blurry. The original city planning could not adapt the
change since more people werepurchasing houses in the suburban area. These
phenomenon caused thedegeneration of natural resources, the loss in the open
space, the aggravationof traffic, the large-scale expenditure of
infrastructure. Scholars pointed outthat a large scale crash down would be
inevitable if the city keep consumingresources in an unsustainable way.

Organizationslike the Environmental Protection Agency offer solutions in many
aspects forthose problems, and finally concluded them into a concept with ten
principlescalled ’smart growth’. After this strategy was published, according
to a survey[1] carried by Smart Growth Network in Oct. 2000, 78% of the US
citizenssupported that it is a necessity to curb the urban crawl.

#### 1.2 Restatement

If we evaluate the progress of thesmart growth, the changes of the ten
principles could be indistinct, but thegoal of the whole activity is clear,
that is make the city a more economicallyprosperous, socially equitable and
environmentally sustainable. When weinterpret the goal of smart growth, we
believe the happiness(in the aspect ofcountry is GNH) would also be affected
by the principles. According to theGeller(2003) [2], smart growth could curb
the damage of the environment,improve the accessibility of infrastructure,
promote the healthy life style andbuild a livable city so that the overall
living quality can be improved. Thus,based on ten principles, we interpret:

• Economically prosperous

• Equitable society

• Environmentally sustainable

• Happiness of citizens as ourreference objects to evaluate the success of the
smart growth.

### 2 Task 1: Model for Smart Growth Measurement

#### 2.1 Outline of Our Approach

We came to an approach to measurethe success of smart growth by treating the
reference objects as the factors inthe process of smart growth:

• Economy

• Equity

• Environment

• Happiness

By analyzing theobjects in order relation, we developed an index named:
Successful Smart Growthindex (SSG)( _Figure 1_ ).

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f1.jpg)
Figure 1: _Framework of SSGindex_

The ten principles are shown in _Table 1_ with corresponding codes from _P1_
to _P10_ :

Code | Principles  
---|---  
P1 | Mix land uses  
P2 | Take advantage of compact building design  
P3 | Create a range of housing opportunities and choices  
P4 | Create walkable neighborhoods  
P5 | Foster distinctive, attractive communities with a strong sense of place  
P6 | Preserve open space, farmland, natural beauty, and critical environmental
areas  
P7 | Strengthen and direct development towards existing communities  
P8 | Provide a variety of transportation choices  
P9 | Make development decisions predictable, fair, and cost effective  
P10 | Encourage community and stakeholder collaboration in development
decisions  
  
Table 1: _Codes of tenprinciples_

Originatingfrom the problem of urban crawl, based on ten principles, different
methods wasoffered to solve it. But if we want to evaluate the progress of the
smartgrowth of a city, it is required to analyze the factors which could be
affectedby the principles, that is our reference objects. Thus we interpret
theprinciples as the influencing factors of the three E0s and happiness.

#### 2.2 Modeling of SSG index

##### 2.2.1 Model assumptions

There are a myraid of factors thatcould affect the process of smart growth, it
is impossible to consider all ofthem, thus we list out the following
assumptions as restrictions of the model:

• We would only consider theactions based on the ten principles, all the
occasionality outside thedefinition of the principles would be ignored.

• As for a developed country, weassume the gap of GNH, EPI, social equity
index between its cities and thecountry itself can be ignored. Thus we could
use the data of the country torepresent the situation in a specific city.

• We consider the total effect ofeach principle to the three E0s and happiness
is the same.

• We assume none of the varianceratio of three E0s and happiness would be
zero.

• We ignore the interactionsamong ten principles as well as the three E0s and
happiness.

##### 2.2.2 Parameter setting

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/t2.jpg)
Table 2: _parameter setting_

##### 2.2.3 The solution to the model

Since different principles mayresult in different effects on the three E0s and
happiness, we set weights tomeasure the progress of smart growth and create a
fomula of SSG index.

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g1.jpg)(1)

In this formula, the success of the smart growth of the city is evaluated by
thevariance ratio of three E0s (∆ _E_ 1 to∆ _E_ 3) and happiness (in the
fomula means ∆ _E_ 4), along with different weights caused by the ten
principles.Different principle plays a different role in the impact to the
three E0s and happiness.[3] For example, as for mix land uses, it would mainly
improve the availabilityof the land which may bring positive economic results.
Thus, the economicfactor values most. Besides, the multi-used land could
improve the livingconvenient level, which is the main factor of happiness.
After that, thedecrese of living radius could improve the environment
situation which givesthe environment the third place while this action may
only present indirecteffect on social equity.

Now we have theranking to the effect on the three E0s and happiness. In the
above example,the ranking order is economy, happiness, environment, equity. In
order toemphasize the principal effect, we used a geometric progession (set
thevariable _r_ be the ranking of theimpact):

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g2.jpg)(2)

The rest nine principles would beanalyzed with the same method as above. _Uji_
is the element

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f2.jpg)  
Figure 2: _the contributionrate of ten principles_

in the j-th row and i-th column of _Figure 2_. Since each principle
posesrelatively different effects on the three E0s and happiness, we have
already gotthe rank of the priority in _Figure 2_.In the assumption we
consider the the total effect of each principle on thethree E0s and happiness
is the same. Thus we define the weight of the smartgrowth _Wi_ is:

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g3.jpg)(3)

After the calculation, we obtain theweight ( _Wi_ ) of the three E0s and
happiness in _Figure 3_ :

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f3.jpg)
Figure 3: _the weight of SSGindex_

Since the SSG index was created to measure the success of the smart growth of
a city, thus, after outputing the result of the index, some analysis could be
used:

• **SSG** > **0** The smart growth of the city is very likely to be a success.

• **SSG** < **0** The city doesnot matches the principles of smart growth.

• **SSG** = **0** The plan of the city attended to one thing and lost sight of
another.

#### 2.3 Evaluation of the Model

To assess whether a model cansolve a problem or not, we need to figure out the
comprehensiveness. In orderto assess the success of the smart growth, we
consulted 4 indexes:

• **GDP** (GrossDomestic Product)

• **Gini Coefficient**

• **EPI** (EnvironmentPerformance Index)

• **GNH** (Gross National Happiness)

Referring tothe contribution rate in _Figure 2_ , weobtain the significance
ordering of the indexes and give out a weighted sum. Asfor the success of
smart growth, we interpret it as the combined effect of our4 reference
objects: Economy, Equity, Environment, Happiness. Thus, the SSGindex includes
all the aspects of smart growth.

##### 2.3.1 Strengths

• Consider the aspects of tenprinciples can not be covered by three E0s, we
chose happiness as the forthparameter to evaluate the success of smart growth.

• During the process of settingthe index, we determine the contribution rate
of ten principles as thereference of deciding the weights of three E0s and
happiness. The complexity of theproblem was simplified because of the decrease
of dimension.

• When we decide the weight ofthree E0s and happiness by analyzing the
contribution rate of tenprinciples, the use of order relations can represent
different influence madeby different principle; the geometric compare scale
shows our good handling onprimary effect and secondary effect.

##### 2.3.2 Weeknesses

• The comparison scale of themodel to set the weight is relatively subjective.

• The number of variable factorsin the model is not enough and the model is
rather simple.

Thus, this model may be affectedby other factors.

### 3 Task 2: Case Study

#### 3.1 The smart growth plan of TAKASAKI City, Japan

From the annual report of TAKASAKICity[4] , we analyzed the actions that could
pose positive effects based on thesmart growth principles. And three E0s and
happiness are the referenceobjects of the principles. Thus, the _Figure4_
focuses on the actions related to the reference objects.

Firstly, as for the reforming of land distribution, it matches **P1**. This
action would create a multi-function community and theutility of land. The
government of TAKASAKI strengthen a lot of infrastructureand the quality of
roads which would greatly improve the convenience of thecitizens and
redistribute the social resources, which matches **P7** , **P4** and **P8**.
In orderto make the environmental situation better, more budgets were invested
tolandscape planning and afforestation. Both of them can be treated as the
partof sustainable development and they match **P6** and **P5**. The
maintenance of parking lots and the restriction on the outdooradvertisements
could give the citizens a more pleasant home. Thus we believethat these
matches **P5**.

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f4.jpg)
Figure 4: _Current smart growthplan of TAKASAKI city_

#### 3.2 The smart growth plan of Mobile City, US

Similarly, we obtained the actionsof the Mobile City in the USA ( _Figure 5_
)from the annual report [5].

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f5.jpg)
Figure 5: _Current smart growthplan of Mobile city_

As for mobile city in the US,enhancing business sevice would make the
companies in the city morecompetitive, thus, it matches **P9**.
Trafficengineering includes the improvement of the road and public
transportation, sothis area would benefit **P4** and **P7**. The landfill
ability is related to **P6**. Morebudget on the public safety area could
improve the sense of safety among thecitizens while the increasing number of
parks could build them a bettercommunity to live in, thus, they match **P5**
and **P7**.

#### 3.3 Assessment of the plan in two cities by SSG index

In the condition when the societyhas spare resources and the salaries are
relatively stable, from the theory ofmultiplier [6], the fiscal expenditure is
positively correlated to economicgrowth. As for a developed country, we assume
the gap of GNH, EPI, socialequity index between the its cities and the country
itself can be ignored. Thuswe could use the data of the country to represent
the situation in a specificcity.[7][8]

##### 3.3.1 Calculation

Referring to our SSG index whichcould measure the success of the smart growth,
we have already got the formulain _equation (1)_.

we needto get a result based on the weights (which has been obtained in
_Figure 3_ and the

variance ratio of the followingreference objects:

• **Economically prosperous** ( _Table 3_ ): Based on the assumption,according
to the theory of multiplier, we could approximately replace thevariance ratio
of the economic growth with the variance ratio of fiscalexpenditure. From the
financial statements of the city, we could get the dataand figure out the
variance ratio of fiscal expenditure.

_FE_ 2 | budget of fiscal expenditure of this year  
---|---  
_FE_ 1 | fiscal expenditure of last year  
Table 3: _Calculation parameterof Economically prosperous_
![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/gg.JPG)(4)

• **Social justice** ( _Table 4_ ): Based on the definition of the Gini
coefficient [9] andsocial justice [10], as well as the completeness of the
Gini coefficient, weuse the variance ratio of Gini coefficient as the
reference index of socialjustice.

_SJ_ 2 | Gini index of this year  
---|---  
_SJ_ 1 | Gini index of last year  
Table 4: _Calculation parameterof Social justice_
![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g4.jpg)(5)

• **Environment sustainable** ( _Table 5_ ): We evaluate the success of
asustainable environment by using the variance ratio of
EnvironmentalPerformance Index[8] as the parameter.

_EPI_ 2 | EPI index of this year  
---|---  
_EPI_ 1 | EPI index of last year  
Table 5: _Calculation parameterof Environment sustainable_
![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g6.jpg)(6)

• **Happiness** ( _Table 6_ ): Based on the definition of the GNH (Gross
NationalHappiness)[11], the measurement of happiness [12] could be judged by
thevariance ratio of GNH.

_H_ 2 | GNH index of this year  
---|---  
_H_ 1 | GNH index of last year  
Table 6: _Calculation parameterof Happiness_
![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g7.jpg)(7)

Using equation _(1)_ , we have _Figure 6_.

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f6.jpg)
Figure 6: _The SSG index of thegrowth plan in TAKASAKI and Mobile City_

##### 3.3.2 Result

In _Figure 6_ , the SSG index of TAKASAKI city is -0.094097012, thus wedefine
this city as **The city does notmatches the principles of smart growth.**

In the MobileCity the result is 0.869158697, thus we define it as **The smart
growth plan the city is very likely to be a success.**

### 4 Task 3 & 4: Our Smart Growth Plan

Based on the ten principles of smartgrowth, we developed a plan with 6
initiatives: (consult _table 1_ about the codes of ten principles)

• build commercial center ( **P1** )

• encourage the green commutingby introducing free bicycles ( **P8** )

• timly restriction of communitytraffic to allow pedestrian-oriented
activities ( **P4** , **P5** )

• direct participation in policymaking in a small range ( **P10** )

• improve medical benefits ( **P7** )

• perfect the infrastructure inthe residential area ( **P7** , **P5** )

#### 4.1 TAKASAKI City

TAKASAKI City is becoming a centerof handicraft industry and trade in Kanto
region [13]. And the city is in themiddle of the traffic network, which would
attract the people to do bussinessactivities. Thus, the increase of number of
the commercial centers would benecessary.

According to therelationship between natural landscape and human [14], we
value the necessityof building a eco-friendly city. Plus, the topography of
the city is gentle.Thus we encourage the green commuting by introducing free
bicycles.

Moreover, tenprinciples emphasize the importance of an attractive community.
So action 3, 5and 6 of our plan were made for it.

Lastly, inorder to have the citizens participate in the administration and
discussion ofstate affairs, we decided to carry out the action of direct
participation inpolicy making in a small range.

#### 4.2 Mobile City

Mobile is a city rich in tourismresources [15]. Thus we believe the increase
number of the commercial centerswould pose positive effect on tourism.
Analyzing the map [16] of the Mobilecity, the topography of the city is
convenient for bicycles introducing.

And there is acarnival in the city called Mardi Gras [17], thus we believe
timly restrictionof community traffic to allow pedestrian-oriented activities
would benecessary.

Following the ten principles,actions 4 and 5 would improve the happiness in
the city.

Moreover, theoccurrence of hurricane in this area [18] is frequent, so the
infrastructure inthe residential area aims to protect the citizens from the
damage of thehurricane.

#### 4.3 A more robust model to evaluatethe individual initiatives: FSGP index

##### 4.3.1 Assumption

• The population factor wouldonly affect economy factor and happiness factor.

• Population factor canapproximately replace all the factors of smart growth
in a city.

##### 4.3.2 Parameter setting

##### 4.3.3 The process of enhancing theSSG model

In order to evaluate whether a planadheres to the smart growth principles or
not, we enhanced the original SSGindex.

For eachinitiative, we need to calculate the contribution to the three E0s and
happiness.And thus, we developed an index named SGP (Smart Growth Plan) based
on SSGindex:

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g8.jpg)(8)

Assume the initiative involves principles _Pj_ $_{1}$ … _Pj_ $_{n}$ (n ≤ 10).
Thus:

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/g9.jpg)(9)

and the value of _Wi_ could be found in _Figure 3_.

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/t7.jpg)
Table 7: _parameter setting_
![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f7.jpg)
Figure 7: _The SGP index of theinitiatives in our smart growth plan_

Using theformula _8_ , we obtain the SGP indexfor each initiative and formed a
table (shown in _Figure 7_ :

Inspired by thetheory of optimum population, we considered the relationship of
populationdensity and GNP, and tried to find out the best population density.
Then, we comparedthe the best population density and the current population
density so that theadjustment coefficient of the adjustment to SSG index.
After that, wecalculated by using the adjustment algebraic model and obtained
a new resultnamed Final Smart Growth Plan (FSGP).

Now we start to optimize the modelof FSGP.

Based on thetheory of optimum population, we decided to use quadratic function
simulationto figure the relation between the population and GNP in the states
of US andcounties in

Japan. And we have the figureshowing the results: ( _Figure 8_ , _Figure 9_ )

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f8.jpg)
Figure 8: _The relation betweenthe population and GNP in the states of US_
![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f9.jpg)
Figure 9: _The relation betweenthe population and GNP in the counties of
Japan_

We determinedthe proper population density for development as the population
density of thestates or counties which is higher than the average GNP. Thus we
used thestraight line: y = average GNP to cut out the figure of the
correspondingcites.( _Figure 10_ , _Figure 11_ ) (shown in next page)

We found outthe straight line and the fitting function intersects in 2 places
(L and H).Andwe figured out the extreme point M. Thus we define the city with
the populationdensity in

![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f10.jpg)
Figure 10: _The relationbetween the population and GNP in the states of US
after using the straightline y = average GNP to cut out_
![img](https://raw.githubusercontent.com/wonggwan/Photo_backup/master/icm/f11.jpg)
Figure11: _The relation between the populationand GNP in the counties of Japan
after using the straight line y = average GNPto cut out_

the interval of L to H (L, H) as thecondition which the economy development is
effective to the smart growth of thecity. The population density of the city
which does not locate in the intervalwould be treated as inapporopriate in
development.

Futhermore,we introduced the limitation factor of population A. With a certain
populationwhich is suitable for city development, a slightly increase in the
populationdensity would bring relatively great GNP effect. When the population
densityalmost approaches the optimized population density, a large scale of
populationdensity increase would only bring slight effect to GNP.

This curve ofthe function of population density and GNP is convex, which
proves thecorrectness of the fitting figure.

Since under theinfluence of population factor, the initiative of the plan on
the economiclevel could not reach the ideal result. Thus, we defined the
limitation factorA as the real number in the interval of [0, 1]. Thus, we
concluded the theregulation of change and adjusted the economy