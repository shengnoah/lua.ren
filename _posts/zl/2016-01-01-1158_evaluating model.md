---
layout: post
title: evaluating model 
tags: [lua文章]
categories: [topic]
---
<style type="text/css">
                            .mjpage .MJX-monospace {
                            font-family: monospace
                            }

                            .mjpage .MJX-sans-serif {
                            font-family: sans-serif
                            }

                            .mjpage {
                            display: inline;
                            font-style: normal;
                            font-weight: normal;
                            line-height: normal;
                            font-size: 100%;
                            font-size-adjust: none;
                            text-indent: 0;
                            text-align: left;
                            text-transform: none;
                            letter-spacing: normal;
                            word-spacing: normal;
                            word-wrap: normal;
                            white-space: nowrap;
                            float: none;
                            direction: ltr;
                            max-width: none;
                            max-height: none;
                            min-width: 0;
                            min-height: 0;
                            border: 0;
                            padding: 0;
                            margin: 0
                            }

                            .mjpage * {
                            transition: none;
                            -webkit-transition: none;
                            -moz-transition: none;
                            -ms-transition: none;
                            -o-transition: none
                            }

                            .mjx-svg-href {
                            fill: blue;
                            stroke: blue
                            }

                            .MathJax_SVG_LineBox {
                            display: table!important
                            }

                            .MathJax_SVG_LineBox span {
                            display: table-cell!important;
                            width: 10000em!important;
                            min-width: 0;
                            max-width: none;
                            padding: 0;
                            border: 0;
                            margin: 0
                            }

                            .mjpage__block {
                            text-align: center;
                            margin: 1em 0em;
                            position: relative;
                            display: block!important;
                            text-indent: 0;
                            max-width: none;
                            max-height: none;
                            min-width: 0;
                            min-height: 0;
                            width: 100%
                            }</style><meta name="generator" content="Hexo 4.2.0"/><h2 id="Summary"><a href="#Summary" class="headerlink" title="Summary"></a>Summary</h2><ul>
<li>High variance(over-fitting)<ul>
<li>More training samples</li>
<li>Less features</li>
<li>Increase <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.355ex" height="2.176ex" style="vertical-align: -0.338ex;" viewBox="0 -791.3 583.5 936.9" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-1-Title">
<title id="MathJax-SVG-1-Title">lambda</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E1-MJMATHI-3BB" d="M166 673Q166 685 183 694H202Q292 691 316 644Q322 629 373 486T474 207T524 67Q531 47 537 34T546 15T551 6T555 2T556 -2T550 -11H482Q457 3 450 18T399 152L354 277L340 262Q327 246 293 207T236 141Q211 112 174 69Q123 9 111 -1T83 -12Q47 -12 47 20Q47 37 61 52T199 187Q229 216 266 252T321 306L338 322Q338 323 288 462T234 612Q214 657 183 657Q166 657 166 673Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E1-MJMATHI-3BB" x="0" y="0"></use>
</g>
</svg></span> / [SVM]Decrease <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.766ex" height="2.176ex" style="vertical-align: -0.338ex;" viewBox="0 -791.3 760.5 936.9" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-2-Title">
<title id="MathJax-SVG-2-Title">C</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E2-MJMATHI-43" d="M50 252Q50 367 117 473T286 641T490 704Q580 704 633 653Q642 643 648 636T656 626L657 623Q660 623 684 649Q691 655 699 663T715 679T725 690L740 705H746Q760 705 760 698Q760 694 728 561Q692 422 692 421Q690 416 687 415T669 413H653Q647 419 647 422Q647 423 648 429T650 449T651 481Q651 552 619 605T510 659Q484 659 454 652T382 628T299 572T226 479Q194 422 175 346T156 222Q156 108 232 58Q280 24 350 24Q441 24 512 92T606 240Q610 253 612 255T628 257Q648 257 648 248Q648 243 647 239Q618 132 523 55T319 -22Q206 -22 128 53T50 252Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E2-MJMATHI-43" x="0" y="0"></use>
</g>
</svg></span></li>
<li>[Neural Network] Less layers, less units</li>
<li>[SVM] Increase <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.33ex" height="1.676ex" style="vertical-align: -0.338ex;" viewBox="0 -576.1 572.5 721.6" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-3-Title">
<title id="MathJax-SVG-3-Title">sigma</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E3-MJMATHI-3C3" d="M184 -11Q116 -11 74 34T31 147Q31 247 104 333T274 430Q275 431 414 431H552Q553 430 555 429T559 427T562 425T565 422T567 420T569 416T570 412T571 407T572 401Q572 357 507 357Q500 357 490 357T476 358H416L421 348Q439 310 439 263Q439 153 359 71T184 -11ZM361 278Q361 358 276 358Q152 358 115 184Q114 180 114 178Q106 141 106 117Q106 67 131 47T188 26Q242 26 287 73Q316 103 334 153T356 233T361 278Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E3-MJMATHI-3C3" x="0" y="0"></use>
</g>
</svg></span></li>
</ul>
</li>
<li>High bias(under-fitting)<ul>
<li>Get additional features</li>
<li>Add polynomial features</li>
<li>Decrease <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.355ex" height="2.176ex" style="vertical-align: -0.338ex;" viewBox="0 -791.3 583.5 936.9" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-4-Title">
<title id="MathJax-SVG-4-Title">lambda</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E4-MJMATHI-3BB" d="M166 673Q166 685 183 694H202Q292 691 316 644Q322 629 373 486T474 207T524 67Q531 47 537 34T546 15T551 6T555 2T556 -2T550 -11H482Q457 3 450 18T399 152L354 277L340 262Q327 246 293 207T236 141Q211 112 174 69Q123 9 111 -1T83 -12Q47 -12 47 20Q47 37 61 52T199 187Q229 216 266 252T321 306L338 322Q338 323 288 462T234 612Q214 657 183 657Q166 657 166 673Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E4-MJMATHI-3BB" x="0" y="0"></use>
</g>
</svg></span> / [SVM]Increase <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.766ex" height="2.176ex" style="vertical-align: -0.338ex;" viewBox="0 -791.3 760.5 936.9" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-5-Title">
<title id="MathJax-SVG-5-Title">C</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E5-MJMATHI-43" d="M50 252Q50 367 117 473T286 641T490 704Q580 704 633 653Q642 643 648 636T656 626L657 623Q660 623 684 649Q691 655 699 663T715 679T725 690L740 705H746Q760 705 760 698Q760 694 728 561Q692 422 692 421Q690 416 687 415T669 413H653Q647 419 647 422Q647 423 648 429T650 449T651 481Q651 552 619 605T510 659Q484 659 454 652T382 628T299 572T226 479Q194 422 175 346T156 222Q156 108 232 58Q280 24 350 24Q441 24 512 92T606 240Q610 253 612 255T628 257Q648 257 648 248Q648 243 647 239Q618 132 523 55T319 -22Q206 -22 128 53T50 252Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E5-MJMATHI-43" x="0" y="0"></use>
</g>
</svg></span></li>
<li>[Neural Network] More layers, more units</li>
<li>[SVM] Decrease <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.33ex" height="1.676ex" style="vertical-align: -0.338ex;" viewBox="0 -576.1 572.5 721.6" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-6-Title">
<title id="MathJax-SVG-6-Title">sigma</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E6-MJMATHI-3C3" d="M184 -11Q116 -11 74 34T31 147Q31 247 104 333T274 430Q275 431 414 431H552Q553 430 555 429T559 427T562 425T565 422T567 420T569 416T570 412T571 407T572 401Q572 357 507 357Q500 357 490 357T476 358H416L421 348Q439 310 439 263Q439 153 359 71T184 -11ZM361 278Q361 358 276 358Q152 358 115 184Q114 180 114 178Q106 141 106 117Q106 67 131 47T188 26Q242 26 287 73Q316 103 334 153T356 233T361 278Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E6-MJMATHI-3C3" x="0" y="0"></use>
</g>
</svg></span></li>
</ul>
</li>
</ul>
<h2 id="Testing"><a href="#Testing" class="headerlink" title="Testing"></a>Testing</h2><p>Testing is used to evaluate how well the model perform.</p>
<p>Split data into <code>Training</code> : <code>Test</code> = 7:3.</p>
<h2 id="Cross-Validation"><a href="#Cross-Validation" class="headerlink" title="Cross Validation"></a>Cross Validation</h2><p>Cross validation is used to use different data set to choose the best model and evaluate it. </p>
<ol>
<li>Split data into <code>Training</code> : <code>Cross Validation</code> : <code>Test</code> = 6:2:2.</li>
<li>Use <code>Cross Validation</code> set to try on different models.</li>
<li>Use <code>Test</code> set to evaluate the best model selected by step 2</li>
</ol>
<h2 id="Learning-Curve"><a href="#Learning-Curve" class="headerlink" title="Learning Curve"></a>Learning Curve</h2><p>Learning curve is to plot <strong>(training set size, <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="7.624ex" height="2.843ex" style="vertical-align: -1.005ex;" viewBox="0 -791.3 3282.4 1223.9" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-7-Title">
<title id="MathJax-SVG-7-Title">J_{training}</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E7-MJMATHI-4A" d="M447 625Q447 637 354 637H329Q323 642 323 645T325 664Q329 677 335 683H352Q393 681 498 681Q541 681 568 681T605 682T619 682Q633 682 633 672Q633 670 630 658Q626 642 623 640T604 637Q552 637 545 623Q541 610 483 376Q420 128 419 127Q397 64 333 21T195 -22Q137 -22 97 8T57 88Q57 130 80 152T132 174Q177 174 182 130Q182 98 164 80T123 56Q115 54 115 53T122 44Q148 15 197 15Q235 15 271 47T324 130Q328 142 387 380T447 625Z"></path>
<path stroke-width="1" id="E7-MJMATHI-74" d="M26 385Q19 392 19 395Q19 399 22 411T27 425Q29 430 36 430T87 431H140L159 511Q162 522 166 540T173 566T179 586T187 603T197 615T211 624T229 626Q247 625 254 615T261 596Q261 589 252 549T232 470L222 433Q222 431 272 431H323Q330 424 330 420Q330 398 317 385H210L174 240Q135 80 135 68Q135 26 162 26Q197 26 230 60T283 144Q285 150 288 151T303 153H307Q322 153 322 145Q322 142 319 133Q314 117 301 95T267 48T216 6T155 -11Q125 -11 98 4T59 56Q57 64 57 83V101L92 241Q127 382 128 383Q128 385 77 385H26Z"></path>
<path stroke-width="1" id="E7-MJMATHI-72" d="M21 287Q22 290 23 295T28 317T38 348T53 381T73 411T99 433T132 442Q161 442 183 430T214 408T225 388Q227 382 228 382T236 389Q284 441 347 441H350Q398 441 422 400Q430 381 430 363Q430 333 417 315T391 292T366 288Q346 288 334 299T322 328Q322 376 378 392Q356 405 342 405Q286 405 239 331Q229 315 224 298T190 165Q156 25 151 16Q138 -11 108 -11Q95 -11 87 -5T76 7T74 17Q74 30 114 189T154 366Q154 405 128 405Q107 405 92 377T68 316T57 280Q55 278 41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E7-MJMATHI-61" d="M33 157Q33 258 109 349T280 441Q331 441 370 392Q386 422 416 422Q429 422 439 414T449 394Q449 381 412 234T374 68Q374 43 381 35T402 26Q411 27 422 35Q443 55 463 131Q469 151 473 152Q475 153 483 153H487Q506 153 506 144Q506 138 501 117T481 63T449 13Q436 0 417 -8Q409 -10 393 -10Q359 -10 336 5T306 36L300 51Q299 52 296 50Q294 48 292 46Q233 -10 172 -10Q117 -10 75 30T33 157ZM351 328Q351 334 346 350T323 385T277 405Q242 405 210 374T160 293Q131 214 119 129Q119 126 119 118T118 106Q118 61 136 44T179 26Q217 26 254 59T298 110Q300 114 325 217T351 328Z"></path>
<path stroke-width="1" id="E7-MJMATHI-69" d="M184 600Q184 624 203 642T247 661Q265 661 277 649T290 619Q290 596 270 577T226 557Q211 557 198 567T184 600ZM21 287Q21 295 30 318T54 369T98 420T158 442Q197 442 223 419T250 357Q250 340 236 301T196 196T154 83Q149 61 149 51Q149 26 166 26Q175 26 185 29T208 43T235 78T260 137Q263 149 265 151T282 153Q302 153 302 143Q302 135 293 112T268 61T223 11T161 -11Q129 -11 102 10T74 74Q74 91 79 106T122 220Q160 321 166 341T173 380Q173 404 156 404H154Q124 404 99 371T61 287Q60 286 59 284T58 281T56 279T53 278T49 278T41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E7-MJMATHI-6E" d="M21 287Q22 293 24 303T36 341T56 388T89 425T135 442Q171 442 195 424T225 390T231 369Q231 367 232 367L243 378Q304 442 382 442Q436 442 469 415T503 336T465 179T427 52Q427 26 444 26Q450 26 453 27Q482 32 505 65T540 145Q542 153 560 153Q580 153 580 145Q580 144 576 130Q568 101 554 73T508 17T439 -10Q392 -10 371 17T350 73Q350 92 386 193T423 345Q423 404 379 404H374Q288 404 229 303L222 291L189 157Q156 26 151 16Q138 -11 108 -11Q95 -11 87 -5T76 7T74 17Q74 30 112 180T152 343Q153 348 153 366Q153 405 129 405Q91 405 66 305Q60 285 60 284Q58 278 41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E7-MJMATHI-67" d="M311 43Q296 30 267 15T206 0Q143 0 105 45T66 160Q66 265 143 353T314 442Q361 442 401 394L404 398Q406 401 409 404T418 412T431 419T447 422Q461 422 470 413T480 394Q480 379 423 152T363 -80Q345 -134 286 -169T151 -205Q10 -205 10 -137Q10 -111 28 -91T74 -71Q89 -71 102 -80T116 -111Q116 -121 114 -130T107 -144T99 -154T92 -162L90 -164H91Q101 -167 151 -167Q189 -167 211 -155Q234 -144 254 -122T282 -75Q288 -56 298 -13Q311 35 311 43ZM384 328L380 339Q377 350 375 354T369 368T359 382T346 393T328 402T306 405Q262 405 221 352Q191 313 171 233T151 117Q151 38 213 38Q269 38 323 108L331 118L384 328Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E7-MJMATHI-4A" x="0" y="0"></use>
<g transform="translate(555,-150)">
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-74" x="0" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-72" x="361" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-61" x="812" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-69" x="1342" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-6E" x="1688" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-69" x="2288" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-6E" x="2634" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E7-MJMATHI-67" x="3234" y="0"></use>
</g>
</g>
</svg></span>)</strong> and <strong>(training set size, <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="4.033ex" height="3.343ex" style="vertical-align: -1.505ex;" viewBox="0 -791.3 1736.6 1439.2" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-8-Title">
<title id="MathJax-SVG-8-Title">J_{cv}</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E8-MJMATHI-4A" d="M447 625Q447 637 354 637H329Q323 642 323 645T325 664Q329 677 335 683H352Q393 681 498 681Q541 681 568 681T605 682T619 682Q633 682 633 672Q633 670 630 658Q626 642 623 640T604 637Q552 637 545 623Q541 610 483 376Q420 128 419 127Q397 64 333 21T195 -22Q137 -22 97 8T57 88Q57 130 80 152T132 174Q177 174 182 130Q182 98 164 80T123 56Q115 54 115 53T122 44Q148 15 197 15Q235 15 271 47T324 130Q328 142 387 380T447 625Z"></path>
<path stroke-width="1" id="E8-MJMATHI-63" d="M34 159Q34 268 120 355T306 442Q362 442 394 418T427 355Q427 326 408 306T360 285Q341 285 330 295T319 325T330 359T352 380T366 386H367Q367 388 361 392T340 400T306 404Q276 404 249 390Q228 381 206 359Q162 315 142 235T121 119Q121 73 147 50Q169 26 205 26H209Q321 26 394 111Q403 121 406 121Q410 121 419 112T429 98T420 83T391 55T346 25T282 0T202 -11Q127 -11 81 37T34 159Z"></path>
<path stroke-width="1" id="E8-MJMATHI-76" d="M173 380Q173 405 154 405Q130 405 104 376T61 287Q60 286 59 284T58 281T56 279T53 278T49 278T41 278H27Q21 284 21 287Q21 294 29 316T53 368T97 419T160 441Q202 441 225 417T249 361Q249 344 246 335Q246 329 231 291T200 202T182 113Q182 86 187 69Q200 26 250 26Q287 26 319 60T369 139T398 222T409 277Q409 300 401 317T383 343T365 361T357 383Q357 405 376 424T417 443Q436 443 451 425T467 367Q467 340 455 284T418 159T347 40T241 -11Q177 -11 139 22Q102 54 102 117Q102 148 110 181T151 298Q173 362 173 380Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E8-MJMATHI-4A" x="0" y="0"></use>
<g transform="translate(555,-265)">
<text font-family="monospace" stroke="none" transform="scale(50.74127551116547) matrix(1 0 0 -1 0 0)"></text>
 <use transform="scale(0.707)" xlink:href="#E8-MJMATHI-63" x="609" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E8-MJMATHI-76" x="1043" y="0"></use>
</g>
</g>
</svg></span>)</strong> so that we could know</p>
<ul>
<li>High bias: High error on both curves. Curves flat out quickly, and get to same error.</li>
<li>High variance: High <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="3.032ex" height="2.509ex" style="vertical-align: -0.671ex;" viewBox="0 -791.3 1305.3 1080.4" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-9-Title">
<title id="MathJax-SVG-9-Title">J_{cv}</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E9-MJMATHI-4A" d="M447 625Q447 637 354 637H329Q323 642 323 645T325 664Q329 677 335 683H352Q393 681 498 681Q541 681 568 681T605 682T619 682Q633 682 633 672Q633 670 630 658Q626 642 623 640T604 637Q552 637 545 623Q541 610 483 376Q420 128 419 127Q397 64 333 21T195 -22Q137 -22 97 8T57 88Q57 130 80 152T132 174Q177 174 182 130Q182 98 164 80T123 56Q115 54 115 53T122 44Q148 15 197 15Q235 15 271 47T324 130Q328 142 387 380T447 625Z"></path>
<path stroke-width="1" id="E9-MJMATHI-63" d="M34 159Q34 268 120 355T306 442Q362 442 394 418T427 355Q427 326 408 306T360 285Q341 285 330 295T319 325T330 359T352 380T366 386H367Q367 388 361 392T340 400T306 404Q276 404 249 390Q228 381 206 359Q162 315 142 235T121 119Q121 73 147 50Q169 26 205 26H209Q321 26 394 111Q403 121 406 121Q410 121 419 112T429 98T420 83T391 55T346 25T282 0T202 -11Q127 -11 81 37T34 159Z"></path>
<path stroke-width="1" id="E9-MJMATHI-76" d="M173 380Q173 405 154 405Q130 405 104 376T61 287Q60 286 59 284T58 281T56 279T53 278T49 278T41 278H27Q21 284 21 287Q21 294 29 316T53 368T97 419T160 441Q202 441 225 417T249 361Q249 344 246 335Q246 329 231 291T200 202T182 113Q182 86 187 69Q200 26 250 26Q287 26 319 60T369 139T398 222T409 277Q409 300 401 317T383 343T365 361T357 383Q357 405 376 424T417 443Q436 443 451 425T467 367Q467 340 455 284T418 159T347 40T241 -11Q177 -11 139 22Q102 54 102 117Q102 148 110 181T151 298Q173 362 173 380Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E9-MJMATHI-4A" x="0" y="0"></use>
<g transform="translate(555,-150)">
 <use transform="scale(0.707)" xlink:href="#E9-MJMATHI-63" x="0" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E9-MJMATHI-76" x="433" y="0"></use>
</g>
</g>
</svg></span> and low <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="7.624ex" height="2.843ex" style="vertical-align: -1.005ex;" viewBox="0 -791.3 3282.4 1223.9" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-10-Title">
<title id="MathJax-SVG-10-Title">J_{training}</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E10-MJMATHI-4A" d="M447 625Q447 637 354 637H329Q323 642 323 645T325 664Q329 677 335 683H352Q393 681 498 681Q541 681 568 681T605 682T619 682Q633 682 633 672Q633 670 630 658Q626 642 623 640T604 637Q552 637 545 623Q541 610 483 376Q420 128 419 127Q397 64 333 21T195 -22Q137 -22 97 8T57 88Q57 130 80 152T132 174Q177 174 182 130Q182 98 164 80T123 56Q115 54 115 53T122 44Q148 15 197 15Q235 15 271 47T324 130Q328 142 387 380T447 625Z"></path>
<path stroke-width="1" id="E10-MJMATHI-74" d="M26 385Q19 392 19 395Q19 399 22 411T27 425Q29 430 36 430T87 431H140L159 511Q162 522 166 540T173 566T179 586T187 603T197 615T211 624T229 626Q247 625 254 615T261 596Q261 589 252 549T232 470L222 433Q222 431 272 431H323Q330 424 330 420Q330 398 317 385H210L174 240Q135 80 135 68Q135 26 162 26Q197 26 230 60T283 144Q285 150 288 151T303 153H307Q322 153 322 145Q322 142 319 133Q314 117 301 95T267 48T216 6T155 -11Q125 -11 98 4T59 56Q57 64 57 83V101L92 241Q127 382 128 383Q128 385 77 385H26Z"></path>
<path stroke-width="1" id="E10-MJMATHI-72" d="M21 287Q22 290 23 295T28 317T38 348T53 381T73 411T99 433T132 442Q161 442 183 430T214 408T225 388Q227 382 228 382T236 389Q284 441 347 441H350Q398 441 422 400Q430 381 430 363Q430 333 417 315T391 292T366 288Q346 288 334 299T322 328Q322 376 378 392Q356 405 342 405Q286 405 239 331Q229 315 224 298T190 165Q156 25 151 16Q138 -11 108 -11Q95 -11 87 -5T76 7T74 17Q74 30 114 189T154 366Q154 405 128 405Q107 405 92 377T68 316T57 280Q55 278 41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E10-MJMATHI-61" d="M33 157Q33 258 109 349T280 441Q331 441 370 392Q386 422 416 422Q429 422 439 414T449 394Q449 381 412 234T374 68Q374 43 381 35T402 26Q411 27 422 35Q443 55 463 131Q469 151 473 152Q475 153 483 153H487Q506 153 506 144Q506 138 501 117T481 63T449 13Q436 0 417 -8Q409 -10 393 -10Q359 -10 336 5T306 36L300 51Q299 52 296 50Q294 48 292 46Q233 -10 172 -10Q117 -10 75 30T33 157ZM351 328Q351 334 346 350T323 385T277 405Q242 405 210 374T160 293Q131 214 119 129Q119 126 119 118T118 106Q118 61 136 44T179 26Q217 26 254 59T298 110Q300 114 325 217T351 328Z"></path>
<path stroke-width="1" id="E10-MJMATHI-69" d="M184 600Q184 624 203 642T247 661Q265 661 277 649T290 619Q290 596 270 577T226 557Q211 557 198 567T184 600ZM21 287Q21 295 30 318T54 369T98 420T158 442Q197 442 223 419T250 357Q250 340 236 301T196 196T154 83Q149 61 149 51Q149 26 166 26Q175 26 185 29T208 43T235 78T260 137Q263 149 265 151T282 153Q302 153 302 143Q302 135 293 112T268 61T223 11T161 -11Q129 -11 102 10T74 74Q74 91 79 106T122 220Q160 321 166 341T173 380Q173 404 156 404H154Q124 404 99 371T61 287Q60 286 59 284T58 281T56 279T53 278T49 278T41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E10-MJMATHI-6E" d="M21 287Q22 293 24 303T36 341T56 388T89 425T135 442Q171 442 195 424T225 390T231 369Q231 367 232 367L243 378Q304 442 382 442Q436 442 469 415T503 336T465 179T427 52Q427 26 444 26Q450 26 453 27Q482 32 505 65T540 145Q542 153 560 153Q580 153 580 145Q580 144 576 130Q568 101 554 73T508 17T439 -10Q392 -10 371 17T350 73Q350 92 386 193T423 345Q423 404 379 404H374Q288 404 229 303L222 291L189 157Q156 26 151 16Q138 -11 108 -11Q95 -11 87 -5T76 7T74 17Q74 30 112 180T152 343Q153 348 153 366Q153 405 129 405Q91 405 66 305Q60 285 60 284Q58 278 41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E10-MJMATHI-67" d="M311 43Q296 30 267 15T206 0Q143 0 105 45T66 160Q66 265 143 353T314 442Q361 442 401 394L404 398Q406 401 409 404T418 412T431 419T447 422Q461 422 470 413T480 394Q480 379 423 152T363 -80Q345 -134 286 -169T151 -205Q10 -205 10 -137Q10 -111 28 -91T74 -71Q89 -71 102 -80T116 -111Q116 -121 114 -130T107 -144T99 -154T92 -162L90 -164H91Q101 -167 151 -167Q189 -167 211 -155Q234 -144 254 -122T282 -75Q288 -56 298 -13Q311 35 311 43ZM384 328L380 339Q377 350 375 354T369 368T359 382T346 393T328 402T306 405Q262 405 221 352Q191 313 171 233T151 117Q151 38 213 38Q269 38 323 108L331 118L384 328Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E10-MJMATHI-4A" x="0" y="0"></use>
<g transform="translate(555,-150)">
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-74" x="0" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-72" x="361" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-61" x="812" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-69" x="1342" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-6E" x="1688" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-69" x="2288" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-6E" x="2634" y="0"></use>
 <use transform="scale(0.707)" xlink:href="#E10-MJMATHI-67" x="3234" y="0"></use>
</g>
</g>
</svg></span>, and gap becomes smaller. Indicating more data could help.</li>
</ul>
<h2 id="Skewed-Classes"><a href="#Skewed-Classes" class="headerlink" title="Skewed Classes"></a>Skewed Classes</h2><p>When the composition of target <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.155ex" height="2.009ex" style="vertical-align: -0.671ex;" viewBox="0 -576.1 497.5 865.1" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-11-Title">
<title id="MathJax-SVG-11-Title">y</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E11-MJMATHI-79" d="M21 287Q21 301 36 335T84 406T158 442Q199 442 224 419T250 355Q248 336 247 334Q247 331 231 288T198 191T182 105Q182 62 196 45T238 27Q261 27 281 38T312 61T339 94Q339 95 344 114T358 173T377 247Q415 397 419 404Q432 431 462 431Q475 431 483 424T494 412T496 403Q496 390 447 193T391 -23Q363 -106 294 -155T156 -205Q111 -205 77 -183T43 -117Q43 -95 50 -80T69 -58T89 -48T106 -45Q150 -45 150 -87Q150 -107 138 -122T115 -142T102 -147L99 -148Q101 -153 118 -160T152 -167H160Q177 -167 186 -165Q219 -156 247 -127T290 -65T313 -9T321 21L315 17Q309 13 296 6T270 -6Q250 -11 231 -11Q185 -11 150 11T104 82Q103 89 103 113Q103 170 138 262T173 379Q173 380 173 381Q173 390 173 393T169 400T158 404H154Q131 404 112 385T82 344T65 302T57 280Q55 278 41 278H27Q21 284 21 287Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E11-MJMATHI-79" x="0" y="0"></use>
</g>
</svg></span> is highly skewed(e.g. 99% of <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="1.155ex" height="2.009ex" style="vertical-align: -0.671ex;" viewBox="0 -576.1 497.5 865.1" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-12-Title">
<title id="MathJax-SVG-12-Title">y</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E12-MJMATHI-79" d="M21 287Q21 301 36 335T84 406T158 442Q199 442 224 419T250 355Q248 336 247 334Q247 331 231 288T198 191T182 105Q182 62 196 45T238 27Q261 27 281 38T312 61T339 94Q339 95 344 114T358 173T377 247Q415 397 419 404Q432 431 462 431Q475 431 483 424T494 412T496 403Q496 390 447 193T391 -23Q363 -106 294 -155T156 -205Q111 -205 77 -183T43 -117Q43 -95 50 -80T69 -58T89 -48T106 -45Q150 -45 150 -87Q150 -107 138 -122T115 -142T102 -147L99 -148Q101 -153 118 -160T152 -167H160Q177 -167 186 -165Q219 -156 247 -127T290 -65T313 -9T321 21L315 17Q309 13 296 6T270 -6Q250 -11 231 -11Q185 -11 150 11T104 82Q103 89 103 113Q103 170 138 262T173 379Q173 380 173 381Q173 390 173 393T169 400T158 404H154Q131 404 112 385T82 344T65 302T57 280Q55 278 41 278H27Q21 284 21 287Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E12-MJMATHI-79" x="0" y="0"></use>
</g>
</svg></span> are the same value), we can get a low error ignoring inputs. So we cannot know whether the model improved or not.</p>
<h3 id="Precision-Recall"><a href="#Precision-Recall" class="headerlink" title="Precision/Recall"></a>Precision/Recall</h3><p>| <strong>Actual True</strong> | <strong>Actual False</strong><br/>—|—|—<br/><strong>Predicted True</strong>|True Positive |False Positive<br/><strong>Predicted False</strong>| False Negative | True Negative<br/>(Note: <span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="5.416ex" height="2.509ex" style="vertical-align: -0.671ex;" viewBox="0 -791.3 2332.1 1080.4" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-13-Title">
<title id="MathJax-SVG-13-Title">y=1</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E13-MJMATHI-79" d="M21 287Q21 301 36 335T84 406T158 442Q199 442 224 419T250 355Q248 336 247 334Q247 331 231 288T198 191T182 105Q182 62 196 45T238 27Q261 27 281 38T312 61T339 94Q339 95 344 114T358 173T377 247Q415 397 419 404Q432 431 462 431Q475 431 483 424T494 412T496 403Q496 390 447 193T391 -23Q363 -106 294 -155T156 -205Q111 -205 77 -183T43 -117Q43 -95 50 -80T69 -58T89 -48T106 -45Q150 -45 150 -87Q150 -107 138 -122T115 -142T102 -147L99 -148Q101 -153 118 -160T152 -167H160Q177 -167 186 -165Q219 -156 247 -127T290 -65T313 -9T321 21L315 17Q309 13 296 6T270 -6Q250 -11 231 -11Q185 -11 150 11T104 82Q103 89 103 113Q103 170 138 262T173 379Q173 380 173 381Q173 390 173 393T169 400T158 404H154Q131 404 112 385T82 344T65 302T57 280Q55 278 41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E13-MJMAIN-3D" d="M56 347Q56 360 70 367H707Q722 359 722 347Q722 336 708 328L390 327H72Q56 332 56 347ZM56 153Q56 168 72 173H708Q722 163 722 153Q722 140 707 133H70Q56 140 56 153Z"></path>
<path stroke-width="1" id="E13-MJMAIN-31" d="M213 578L200 573Q186 568 160 563T102 556H83V602H102Q149 604 189 617T245 641T273 663Q275 666 285 666Q294 666 302 660V361L303 61Q310 54 315 52T339 48T401 46H427V0H416Q395 3 257 3Q121 3 100 0H88V46H114Q136 46 152 46T177 47T193 50T201 52T207 57T213 61V578Z"></path>
</defs>
<g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)" aria-hidden="true">
 <use xlink:href="#E13-MJMATHI-79" x="0" y="0"></use>
 <use xlink:href="#E13-MJMAIN-3D" x="775" y="0"></use>
 <use xlink:href="#E13-MJMAIN-31" x="1831" y="0"></use>
</g>
</svg></span> is the <strong><em>rare</em></strong> class that we want to detect)</p>
<ul>
<li><span class="mjpage"><svg xmlns:xlink="http://www.w3.org/1999/xlink" width="26.955ex" height="3.843ex" style="vertical-align: -1.338ex;" viewBox="0 -1078.4 11605.7 1654.5" role="img" focusable="false" xmlns="http://www.w3.org/2000/svg" aria-labelledby="MathJax-SVG-14-Title">
<title id="MathJax-SVG-14-Title">Precision=frac{True Positive}{All Predicted True}</title>
<defs aria-hidden="true">
<path stroke-width="1" id="E14-MJMATHI-50" d="M287 628Q287 635 230 637Q206 637 199 638T192 648Q192 649 194 659Q200 679 203 681T397 683Q587 682 600 680Q664 669 707 631T751 530Q751 453 685 389Q616 321 507 303Q500 302 402 301H307L277 182Q247 66 247 59Q247 55 248 54T255 50T272 48T305 46H336Q342 37 342 35Q342 19 335 5Q330 0 319 0Q316 0 282 1T182 2Q120 2 87 2T51 1Q33 1 33 11Q33 13 36 25Q40 41 44 43T67 46Q94 46 127 49Q141 52 146 61Q149 65 218 339T287 628ZM645 554Q645 567 643 575T634 597T609 619T560 635Q553 636 480 637Q463 637 445 637T416 636T404 636Q391 635 386 627Q384 621 367 550T332 412T314 344Q314 342 395 342H407H430Q542 342 590 392Q617 419 631 471T645 554Z"></path>
<path stroke-width="1" id="E14-MJMATHI-72" d="M21 287Q22 290 23 295T28 317T38 348T53 381T73 411T99 433T132 442Q161 442 183 430T214 408T225 388Q227 382 228 382T236 389Q284 441 347 441H350Q398 441 422 400Q430 381 430 363Q430 333 417 315T391 292T366 288Q346 288 334 299T322 328Q322 376 378 392Q356 405 342 405Q286 405 239 331Q229 315 224 298T190 165Q156 25 151 16Q138 -11 108 -11Q95 -11 87 -5T76 7T74 17Q74 30 114 189T154 366Q154 405 128 405Q107 405 92 377T68 316T57 280Q55 278 41 278H27Q21 284 21 287Z"></path>
<path stroke-width="1" id="E14-MJMATHI-65" d="M39 168Q39 225 58 272T107 350T174 402T244 433T307 442H310Q355 442 388 420T421 355Q421 265 310 237Q261 224 176 223Q139 223 138 221Q138 219 132 186T125 128Q125 81 146 54T209 26T302 45T394 111Q403 121 406 121Q410 121 419 112T429 98T420 82T390 55T344 24T281 -1T205 -11Q126 -11 83 42T39 168ZM373 353Q367 405 305 405Q272 405 244 391T199 357T170 316T154 280T149 261Q149 260 169