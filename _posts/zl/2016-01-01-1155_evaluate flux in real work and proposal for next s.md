---
layout: post
title: evaluate flux in real work and proposal for next step 
tags: [lua文章]
categories: [topic]
---
<p>I used Flux in recent work and after some evaluation, we decide to use it for future development.</p>
<h2 id="1-Intro"><a href="#1-Intro" class="headerlink" title="1. Intro"></a>1. Intro</h2><h3 id="1-1-React-and-Flux"><a href="#1-1-React-and-Flux" class="headerlink" title="1.1 React and Flux"></a>1.1 React and Flux</h3><p><a href="https://facebook.github.io/react/docs/tutorial.html" target="_blank" rel="external noopener noreferrer">React</a> is a library to build <strong>view</strong>. You can build abstract UI components in a declarative way, start with basic ones and finally compose a complex UI. React component is state machine, the whole UI is predictable according to the state. </p>
<p><a href="https://facebook.github.io/flux/docs/overview.html" target="_blank" rel="external noopener noreferrer">Flux</a> is an application architecture, which defines the way of organizing code and data flow. In Facebook’s example implementation, <em>Store</em> is the data, <em>View</em> is the components. <em>View</em> will render based on <em>Store</em>, and <em>Actions</em> can be triggered from <em>View</em> to update <em>Store</em>. </p>
<p><img src="!--￼1--&gt;/images/flux-flow.jpg" alt="flux flow" title="flux flow"/></p>
<p>From <em>Store</em> to <em>View</em> to <em>Store</em>, the data flows only in one direction. It makes the application predictable. Every change in <em>Store</em> will re-render the whole app and it’s still fast thanks to virtual dom.</p>

<h3 id="1-2-ES6"><a href="#1-2-ES6" class="headerlink" title="1.2 ES6"></a>1.2 ES6</h3><h4 id="1-2-1-Intro"><a href="#1-2-1-Intro" class="headerlink" title="1.2.1 Intro"></a>1.2.1 Intro</h4><p>The JavaScript programming language is standardized by ECMA (a standards body like W3C) under the name ECMAScript. ES6 aka ECMAScript 6 or 2015, is a ECMAScript standard approved in the summer of 2015.</p>
<h4 id="1-2-1-Compatibility"><a href="#1-2-1-Compatibility" class="headerlink" title="1.2.1 Compatibility"></a>1.2.1 Compatibility</h4><p>It is basically not supported by all browsers (2015.10), check the <a href="https://kangax.github.io/compat-table/es6/" target="_blank" rel="external noopener noreferrer">compatibility</a>.</p>
<h4 id="1-2-2-Why-learn-it"><a href="#1-2-2-Why-learn-it" class="headerlink" title="1.2.2 Why learn it?"></a>1.2.2 Why learn it?</h4><ul>
<li>It’s the trend, and most new JS frameworks use it.</li>
<li>react recommend writing components using ES6 class, and mixin will be removed and [Higher-Order Components] (<a href="http://jamesknelson.com/structuring-react-applications-higher-order-components/" target="_blank" rel="external noopener noreferrer">http://jamesknelson.com/structuring-react-applications-higher-order-components/</a>) should be used.</li>
</ul>
<h4 id="1-2-3-How-to-use-it"><a href="#1-2-3-How-to-use-it" class="headerlink" title="1.2.3 How to use it?"></a>1.2.3 How to use it?</h4><p>We need to convert ES6 code to ES5 code. For example, use module bundler to convert and bundle the JS files (<code>Webpack</code> + <code>babel-loader</code>).</p>
<p>I am working on a [cheatsheet for ES6] (/learnES6/).<br/><a href="https://github.com/lukehoban/es6features" target="_blank" rel="external noopener noreferrer">ES6 features</a></p>
<h2 id="2-Evaluate-flux-in-real-work"><a href="#2-Evaluate-flux-in-real-work" class="headerlink" title="2 Evaluate flux in real work"></a>2 Evaluate flux in real work</h2><h3 id="2-1-Summary"><a href="#2-1-Summary" class="headerlink" title="2.1 Summary"></a>2.1 Summary</h3><p><strong>Work</strong>: <code>pivot viewer</code> is built using Flux, the implementation is based on Facebook’s example.</p>
<p><strong>Conclusion</strong>: using Flux can reduce bugs and make the development easier in the following aspects:</p>
<ul>
<li>modularize UI components. Each component’s interactions are restricted to itself, and external references should be explicitly referenced.</li>
<li>separation of logic. Components only care about what data they get (props) and what action will be triggered by which interaction. Updating data logic is in Store.</li>
<li>declarative way to build components. One direction data flow makes the interaction more clear.</li>
</ul>
<p>Another benefit could be a better way to organize style sheet, to easily find which style is applied to which component.</p>
<p>While on the other hand, <strong>difficulties</strong>:</p>
<ul>
<li>many Flux implementations and no best practice yet. [Facebook itself has different implementations internally.] (<a href="https://discuss.reactjs.org/t/lost-on-what-flux-implementation-to-use/545/12" target="_blank" rel="external noopener noreferrer">https://discuss.reactjs.org/t/lost-on-what-flux-implementation-to-use/545/12</a>).</li>
<li>learn: <ul>
<li>React </li>
<li>JS (CommonJS format, ES6)</li>
<li>module bundler (<code>Browserify</code>, <code>Webpack</code>), organize JS and style sheet</li>
<li>new test API ()</li>
</ul>
</li>
<li>building form-like widget which contains data update and re-render is complex (maybe solvable)</li>
<li>rewrite existing widgets (wrap kendo widget using react, etc)</li>
<li>some widgets that manipulate dom is not supported in react</li>
<li>the performance is bad when you need to re-render a lot components without reusing (e.g. table cells). (should be solvable)</li>
</ul>
<h3 id="2-2-Current-status"><a href="#2-2-Current-status" class="headerlink" title="2.2 Current status"></a>2.2 Current status</h3><p>The project is a Java Maven project, a multi-page application in JSP. JS is written in plain IIFE style. CSS is not organized either.</p>
<p>For <code>pivot viewer</code>:</p>
<ul>
<li>JS is written in ES5, CommonJS style (simply add <code>export</code> in JS file)</li>
<li>React 0.13, using <code>React.createClass</code> and mixin</li>
<li><code>frontend-maven-plugin</code> will call <code>npm</code> during Maven build phase <code>generate-resources</code>, which calls <code>Browserify</code> to bundle the JS files</li>
<li><code>wro4j</code> will reference the JS file in JSP file, it will also minify the file and append hash to the URL</li>
<li>styles for components are placed in different files, referenced in JSP.</li>
<li>no test cases</li>
</ul>
<h3 id="2-3-Next-step"><a href="#2-3-Next-step" class="headerlink" title="2.3 Next step"></a>2.3 Next step</h3><p><strong>The first step</strong> should be reorganizing the files and use <code>Webpack</code> to bundle file, because:</p>
<ul>
<li>we need to separate different entry point if there will be more pages</li>
<li><code>Webpack</code> support multi entry points (each page has a entry point)</li>
<li><code>Webpack</code> can support ES6 and other module standard </li>
<li><code>Webpack</code> can bundle style sheets</li>
<li><code>Webpack</code> has some powerful features like <code>hot module reload</code> for development</li>
</ul>
<p><strong>The second step</strong> is to reorganize style sheets and use SASS (file extension is <strong>.scss</strong>) , because:</p>
<ul>
<li>CSS files and class names are messy </li>
<li>SASS is more powerful than CSS, especially for code reuse</li>
</ul>
<p>One decision to be made: <em>inline or external CSS?</em></p>
<ul>
<li>inline: style sheet is inserted to source code in <code>style</code> tag. This means no caching in browser.</li>
<li>external CSS: one CSS file for one entry point, need to be referenced in JSP. This will add one more HTTP request.</li>
<li>In both ways, the style sheet is global scope. Still not perfect.</li>
</ul>
<p>The above two steps has been validated to be working in my test, and it will take one or two hour to apply and test it. </p>
<p><strong>Further steps:</strong></p>
<ul>
<li>add test cases</li>
<li>rewrite components following ES6 and React 0.14</li>
<li>refactoring, since some of them are not React, some logic should not be in <em>Store</em></li>
</ul>
<p><strong>Further further steps</strong> or in the mean time:</p>
<ul>
<li>evaluate Redux, since it’s highly recommended, like [here] (<a href="http://jamesknelson.com/which-flux-implementation-should-i-use-with-react/" target="_blank" rel="external noopener noreferrer">http://jamesknelson.com/which-flux-implementation-should-i-use-with-react/</a>) and [here] (<a href="https://github.com/rackt/redux" target="_blank" rel="external noopener noreferrer">https://github.com/rackt/redux</a>)</li>
<li>evaluate [CSS modules] (<a href="https://github.com/CSS-modules/CSS-modules" target="_blank" rel="external noopener noreferrer">https://github.com/CSS-modules/CSS-modules</a>), which is modular, reusable, using no global scope; but a bit complex to use.</li>
<li>build a style guide</li>
<li>use <code>hot module reload</code> in development</li>
</ul>
<h3 id="2-4-Testing"><a href="#2-4-Testing" class="headerlink" title="2.4 Testing"></a>2.4 Testing</h3><h4 id="2-4-1-JSET"><a href="#2-4-1-JSET" class="headerlink" title="2.4.1 JSET"></a>2.4.1 JSET</h4><p><a href="https://facebook.github.io/jest/docs/tutorial-react.html#content" target="_blank" rel="external noopener noreferrer">JEST</a> is a unit test framework promoted by Facebook. It’s:</p>
<blockquote>
<ul>
<li>Built on top of the Jasmine test framework, using familiar expect(value).toBe(other) assertions</li>
<li>Automatically mocks CommonJS modules returned by require(), making most existing code testable</li>
<li>DOM APIs are mocked and tests run in parallel via a small node.js command line utility</li>
</ul>
</blockquote>
<p>It can unit test single component and automatically mock others, but we have some issues:</p>
<ul>
<li>slow, a couple seconds for a test case in watch mode</li>
<li>Kendo UI is referenced in HTML globally, which can not be imported/required as a module. It’s OK when bundling the JS files and visit the web page, but in a unit test, kendo UI needs to be imported somehow. There is no support for npm package for Kendo UI.</li>
<li>no DEBUG in browser [yet] (<a href="https://github.com/facebook/jest/issues/139" target="_blank" rel="external noopener noreferrer">https://github.com/facebook/jest/issues/139</a>).</li>
</ul>
<p>What’s more, it requires some learning for <a href="https://facebook.github.io/react/docs/test-utils.html" target="_blank" rel="external noopener noreferrer"><code>ReactTestUtils.Simulate</code> API</a>, which is used to simulate events.</p>
<h4 id="2-4-2-Karma-plus-karma-webpack"><a href="#2-4-2-Karma-plus-karma-webpack" class="headerlink" title="2.4.2 Karma plus karma-webpack"></a>2.4.2 Karma plus karma-webpack</h4><p>The proposal is using <code>karma</code> with <code>karma-webpack</code>, which:</p>
<ul>
<li>is faster than JEST</li>
<li>can add global reference like Kendo UI using <code>Karma</code></li>
<li>DEBUG in browser</li>
</ul>
<h2 id="3-Plan"><a href="#3-Plan" class="headerlink" title="3 Plan"></a>3 Plan</h2><p>After some discussion, this is the final plan:</p>
<h3 id="3-1-Folder-structure"><a href="#3-1-Folder-structure" class="headerlink" title="3.1 Folder structure"></a>3.1 Folder structure</h3><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/><span class="line">13</span><br/><span class="line">14</span><br/><span class="line">15</span><br/><span class="line">16</span><br/><span class="line">17</span><br/><span class="line">18</span><br/><span class="line">19</span><br/><span class="line">20</span><br/><span class="line">21</span><br/><span class="line">22</span><br/></pre></td><td class="code"><pre><span class="line">|-build            Output folder for bundled JS and CSS file </span><br/><span class="line">|---pivot.js</span><br/><span class="line">|---pivot.css</span><br/><span class="line">|-components       components can be reused (those who do not directly connected to Store)</span><br/><span class="line">|---kendo          kendo components rewritten</span><br/><span class="line">|-dispatcher       reference dispatcher by Facebook (do not touch)</span><br/><span class="line">|-lib              JS utilities (parse data, build pie chart using d3)</span><br/><span class="line">|-mixin            mixin</span><br/><span class="line">|-services         JS services, communicate with API, ajax calls</span><br/><span class="line">|-styles           global styles can be referenced by others</span><br/><span class="line">|---variables.scss define variables</span><br/><span class="line">|---helper.scss    define helper classes, like hide, float etc.</span><br/><span class="line">|---...            </span><br/><span class="line">|-pages            folder contain all pages, each page has a entry JS file and a folder</span><br/><span class="line">|---pivot          single page folder, containing corresponding actions, components, constants and stores</span><br/><span class="line">|-----pivot.js     entry point for pivot page </span><br/><span class="line">|-----actions</span><br/><span class="line">|-----components</span><br/><span class="line">|-------List.jsx   a component named &#39;List&#39;</span><br/><span class="line">|-------List.scss  style for &#39;List&#39;  </span><br/><span class="line">|-----constants</span><br/><span class="line">|-----stores</span><br/></pre></td></tr></tbody></table></figure>
<h3 id="3-2-Work-flow"><a href="#3-2-Work-flow" class="headerlink" title="3.2 Work flow"></a>3.2 Work flow</h3><p>Create a new page called <code>SuperViewer</code>:</p>
<ul>
<li>create new folder <code>SuperViewer</code> and entry point <code>SuperViewer.js</code> under folder <code>pages</code>.</li>
<li>create components, actions, constants, stores, styles, etc.</li>
<li>for a component, reference required style sheet in JS file.</li>
<li>add entry point <code>SuperViewer.js</code> in <code>/app-web/webpack.config.js</code></li>
<li>for development, execute <code>npm run dev</code> in terminal (<code>/app-web</code>), Webpack will watch the changes and automatically re-bundle.</li>
<li>for production, execute <code>npm run prod</code> in terminal, files will be bundled</li>
<li><p>for testing, execute <code>npm run test-flux</code> for simple testing (executed in build process). Execute <code>npm run test-flux-debug</code> for debugging.</p>
</li>
<li><p>[package.json] (../files/package.json)</p>
</li>
<li>[webpack.config.js] (../files/webpack.config.js)</li>
<li>[pom.xml] (../files/pom.xml)</li>
<li>[karma.flux.conf.js] (../files/karma.flux.conf.js)</li>
<li>[karma.flux.ci.conf.js] (../files/karma.flux.ci.conf.js)</li>
</ul>
<h3 id="3-3-TODO"><a href="#3-3-TODO" class="headerlink" title="3.3 TODO"></a>3.3 TODO</h3><ul>
<li>upgrade to React 0.14</li>
<li>build style guide, naming conventions</li>
<li>rewrite components with ES6 syntax and style guide</li>
<li>refactoring code:<ul>
<li>separate logic from store</li>
<li>make more component reusable</li>
</ul>
</li>
<li>investigate building complex React component<ul>
<li>form</li>
<li>large grid</li>
</ul>
</li>
<li>write test cases</li>
<li>Redux seems nice</li>
</ul>
<h3 id="3-3-Style-guide"><a href="#3-3-Style-guide" class="headerlink" title="3.3 Style guide"></a>3.3 Style guide</h3><p>This is some references, we need to build our own:</p>
<ul>
<li>React/JSX: [Airbnb React/JSX Style Guide] (<a href="https://github.com/airbnb/javascript/tree/master/react" target="_blank" rel="external noopener noreferrer">https://github.com/airbnb/javascript/tree/master/react</a>)</li>
<li>CSS/SASS: [Airbnb CSS / Sass Styleguide] (<a href="https://github.com/airbnb/CSS" target="_blank" rel="external noopener noreferrer">https://github.com/airbnb/CSS</a>)</li>
<li>CSS naming: [oocss and bem?] (<a href="https://github.com/airbnb/CSS#oocss-and-bem" target="_blank" rel="external noopener noreferrer">https://github.com/airbnb/CSS#oocss-and-bem</a>) </li>
</ul>