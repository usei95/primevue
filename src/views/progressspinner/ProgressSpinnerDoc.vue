<template>
	<div class="content-section documentation">
		<TabView>
			<TabPanel header="Documentation">
				<h3>Import</h3>
<CodeHighlight lang="javascript">
import ProgressSpinner from 'primevue/progressspinner';
</CodeHighlight>

				<h3>Getting Started</h3>
				<p>ProgressSpinner is defined using ProgressSpinner element.</p>
<CodeHighlight>
<template v-pre>
&lt;ProgressSpinner /&gt;
</template>
</CodeHighlight>

                <h3>Colors</h3>
                <p>Colors of the spinner can be changed by overriding the keyframes animation.</p>
<CodeHighlight lang="css">
@keyframes ui-progress-spinner-color {
    100%,
    0% {
        stroke: #d62d20;
    }
    40% {
        stroke: #0057e7;
    }
    66% {
        stroke: #008744;
    }
    80%,
    90% {
        stroke: #ffa700;
    }
}
</CodeHighlight>

				<h3>Properties</h3>
                <p>Any attribute such as style and class are passed to the main container element. Following are the additional properties to configure the component.</p>
				<div class="doc-tablewrapper">
					<table class="doc-table">
						<thead>
                            <tr>
                                <th>Name</th>
                                <th>Type</th>
                                <th>Default</th>
                                <th>Description</th>
                            </tr>
						</thead>
						<tbody>
                            <tr>
                                <td>strokeWidth</td>
                                <td>string</td>
                                <td>2</td>
                                <td>Width of the circle stroke.</td>
                            </tr>
                            <tr>
                                <td>fill</td>
                                <td>string</td>
                                <td>null</td>
                                <td>Color for the background of the circle.</td>
                            </tr>
                            <tr>
                                <td>animationDuration</td>
                                <td>string</td>
                                <td>2s</td>
                                <td>Duration of the rotate animation.</td>
                            </tr>
						</tbody>
					</table>
				</div>

				<h3>Styling</h3>
				<p>Following is the list of structural style classes, for theming classes visit <router-link to="/theming">theming</router-link> page.</p>
				<div class="doc-tablewrapper">
					<table class="doc-table">
						<thead>
                            <tr>
                                <th>Name</th>
                                <th>Element</th>
                            </tr>
						</thead>
						<tbody>
                            <tr>
                                <td>p-progress-spinner</td>
                                <td>Container element.</td>
                            </tr>
                            <tr>
                                <td>p-progress-circle</td>
                                <td>SVG element.</td>
                            </tr>
                            <tr>
                                <td>p-progress-path</td>
                                <td>Circle element.</td>
                            </tr>
						</tbody>
					</table>
				</div>

				<h3>Dependencies</h3>
				<p>None.</p>
			</TabPanel>

			<TabPanel header="Source">
				<a href="https://github.com/primefaces/primevue/tree/master/src/views/progressspinner" class="btn-viewsource" target="_blank" rel="noopener noreferrer">
					<span>View on GitHub</span>
				</a>
<CodeHighlight>
<template v-pre>
&lt;h3&gt;Basic&lt;/h3&gt;
&lt;ProgressSpinner /&gt;

&lt;h3&gt;Custom&lt;/h3&gt;
&lt;ProgressSpinner style="width:50px;height:50px" strokeWidth="8" fill="#EEEEEE" animationDuration=".5s"/&gt;
</template>
</CodeHighlight>
			</TabPanel>
		</TabView>
	</div>
</template>