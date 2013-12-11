This past week, while researching for another post about IE8 and responsive
design I'm thinking about writing, I pulled stats from a large number of
Google Analytics profiles on all of the widths of the screens, and created a
histogram.

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 260" width="800" height="260" style="max-width:100%;">
	<style>
		rect {
			fill: #999;
		}
		text {
			text-anchor: middle;
			font-size: 12px;
			font-family: sans-serif;
		}
	</style>
	<rect x="0" y="199.9984" width="4" height="0.0016" />
	<rect x="4" y="199.9998" width="4" height="0.0002" />
	<rect x="8" y="199.9397" width="4" height="0.0603" />
	<rect x="32" y="199.9999" width="4" height="0.0001" />
	<rect x="40" y="199.9998" width="4" height="0.0002" />
	<rect x="44" y="199.9996" width="4" height="0.0004" />
	<rect x="48" y="199.4973" width="4" height="0.5027" />
	<rect x="52" y="199.9412" width="4" height="0.0588" />
	<rect x="56" y="199.9994" width="4" height="0.0006" />
	<rect x="60" y="199.9926" width="4" height="0.0074" />
	<rect x="64" y="199.9988" width="4" height="0.0012" />
	<rect x="68" y="199.9769" width="4" height="0.0231" />
	<rect x="72" y="199.9745" width="4" height="0.0255" />
	<rect x="76" y="199.9981" width="4" height="0.0019" />
	<rect x="80" y="199.9992" width="4" height="0.0008" />
	<rect x="84" y="199.9948" width="4" height="0.0052" />
	<rect x="88" y="199.9751" width="4" height="0.0249" />
	<rect x="92" y="199.8974" width="4" height="0.1026" />
	<rect x="96" y="199.8947" width="4" height="0.1053" />
	<rect x="100" y="199.9955" width="4" height="0.0045" />
	<rect x="104" y="199.9984" width="4" height="0.0016" />
	<rect x="108" y="199.9961" width="4" height="0.0039" />
	<rect x="112" y="199.9897" width="4" height="0.0103" />
	<rect x="116" y="199.9815" width="4" height="0.0185" />
	<rect x="120" y="199.9877" width="4" height="0.0123" />
	<rect x="124" y="199.9340" width="4" height="0.0660" />
	<rect x="128" y="175.6942" width="4" height="24.3058" />
	<rect x="132" y="199.9996" width="4" height="0.0004" />
	<rect x="136" y="199.9820" width="4" height="0.0180" />
	<rect x="140" y="199.9775" width="4" height="0.0225" />
	<rect x="144" y="198.3329" width="4" height="1.6671" />
	<rect x="148" y="199.9285" width="4" height="0.0715" />
	<rect x="152" y="199.9673" width="4" height="0.0327" />
	<rect x="156" y="199.9980" width="4" height="0.0020" />
	<rect x="160" y="199.9698" width="4" height="0.0302" />
	<rect x="164" y="199.9996" width="4" height="0.0004" />
	<rect x="168" y="199.9984" width="4" height="0.0016" />
	<rect x="172" y="199.9941" width="4" height="0.0059" />
	<rect x="176" y="199.9984" width="4" height="0.0016" />
	<rect x="180" y="199.9982" width="4" height="0.0018" />
	<rect x="184" y="199.9991" width="4" height="0.0009" />
	<rect x="188" y="199.9979" width="4" height="0.0021" />
	<rect x="192" y="198.5551" width="4" height="1.4449" />
	<rect x="196" y="199.9983" width="4" height="0.0017" />
	<rect x="200" y="199.9978" width="4" height="0.0022" />
	<rect x="204" y="199.9977" width="4" height="0.0023" />
	<rect x="208" y="199.9989" width="4" height="0.0011" />
	<rect x="212" y="199.7888" width="4" height="0.2112" />
	<rect x="216" y="197.9806" width="4" height="2.0194" />
	<rect x="220" y="199.9976" width="4" height="0.0024" />
	<rect x="224" y="199.9990" width="4" height="0.0010" />
	<rect x="228" y="199.9474" width="4" height="0.0526" />
	<rect x="232" y="199.9990" width="4" height="0.0010" />
	<rect x="236" y="199.9975" width="4" height="0.0025" />
	<rect x="240" y="199.5178" width="4" height="0.4822" />
	<rect x="244" y="199.9956" width="4" height="0.0044" />
	<rect x="248" y="199.9913" width="4" height="0.0087" />
	<rect x="252" y="199.9986" width="4" height="0.0014" />
	<rect x="256" y="199.7728" width="4" height="0.2272" />
	<rect x="260" y="199.9990" width="4" height="0.0010" />
	<rect x="264" y="199.9982" width="4" height="0.0018" />
	<rect x="268" y="199.9984" width="4" height="0.0016" />
	<rect x="272" y="199.9637" width="4" height="0.0363" />
	<rect x="276" y="199.9987" width="4" height="0.0013" />
	<rect x="280" y="199.9969" width="4" height="0.0031" />
	<rect x="284" y="199.9951" width="4" height="0.0049" />
	<rect x="288" y="197.2132" width="4" height="2.7868" />
	<rect x="292" y="199.9936" width="4" height="0.0064" />
	<rect x="296" y="199.9795" width="4" height="0.0205" />
	<rect x="300" y="199.9910" width="4" height="0.0090" />
	<rect x="304" y="199.9917" width="4" height="0.0083" />
	<rect x="308" y="188.0553" width="4" height="11.9447" />
	<rect x="312" y="199.9864" width="4" height="0.0136" />
	<rect x="316" y="199.9909" width="4" height="0.0091" />
	<rect x="320" y="198.3586" width="4" height="1.6414" />
	<rect x="324" y="199.9833" width="4" height="0.0167" />
	<rect x="328" y="199.8742" width="4" height="0.1258" />
	<rect x="332" y="199.9829" width="4" height="0.0171" />
	<rect x="336" y="199.9669" width="4" height="0.0331" />
	<rect x="340" y="199.7341" width="4" height="0.2659" />
	<rect x="344" y="199.9777" width="4" height="0.0223" />
	<rect x="348" y="199.9700" width="4" height="0.0300" />
	<rect x="352" y="199.9252" width="4" height="0.0748" />
	<rect x="356" y="199.9717" width="4" height="0.0283" />
	<rect x="360" y="199.8970" width="4" height="0.1030" />
	<rect x="364" y="199.9124" width="4" height="0.0876" />
	<rect x="368" y="199.9408" width="4" height="0.0592" />
	<rect x="372" y="199.9886" width="4" height="0.0114" />
	<rect x="376" y="199.9093" width="4" height="0.0907" />
	<rect x="380" y="199.9347" width="4" height="0.0653" />
	<rect x="384" y="199.2918" width="4" height="0.7082" />
	<rect x="388" y="199.9636" width="4" height="0.0364" />
	<rect x="392" y="199.7747" width="4" height="0.2253" />
	<rect x="396" y="199.6241" width="4" height="0.3759" />
	<rect x="400" y="199.2129" width="4" height="0.7871" />
	<rect x="404" y="198.7070" width="4" height="1.2930" />
	<rect x="408" y="86.3709" width="4" height="113.6291" />
	<rect x="412" y="199.6632" width="4" height="0.3368" />
	<rect x="416" y="199.7094" width="4" height="0.2906" />
	<rect x="420" y="198.9688" width="4" height="1.0312" />
	<rect x="424" y="199.3874" width="4" height="0.6126" />
	<rect x="428" y="196.4782" width="4" height="3.5218" />
	<rect x="432" y="196.6728" width="4" height="3.3272" />
	<rect x="436" y="192.7323" width="4" height="7.2677" />
	<rect x="440" y="199.4304" width="4" height="0.5696" />
	<rect x="444" y="199.5308" width="4" height="0.4692" />
	<rect x="448" y="197.7538" width="4" height="2.2462" />
	<rect x="452" y="199.8439" width="4" height="0.1561" />
	<rect x="456" y="196.7335" width="4" height="3.2665" />
	<rect x="460" y="188.9340" width="4" height="11.0660" />
	<rect x="464" y="199.6371" width="4" height="0.3629" />
	<rect x="468" y="198.6225" width="4" height="1.3775" />
	<rect x="472" y="199.5870" width="4" height="0.4130" />
	<rect x="476" y="196.8799" width="4" height="3.1201" />
	<rect x="480" y="199.2169" width="4" height="0.7831" />
	<rect x="484" y="199.4903" width="4" height="0.5097" />
	<rect x="488" y="199.4931" width="4" height="0.5069" />
	<rect x="492" y="198.0626" width="4" height="1.9374" />
	<rect x="496" y="197.3957" width="4" height="2.6043" />
	<rect x="500" y="197.5657" width="4" height="2.4343" />
	<rect x="504" y="199.6822" width="4" height="0.3178" />
	<rect x="508" y="199.9198" width="4" height="0.0802" />
	<rect x="512" y="12.1357" width="4" height="187.8643" />
	<rect x="516" y="199.5375" width="4" height="0.4625" />
	<rect x="520" y="197.4637" width="4" height="2.5363" />
	<rect x="524" y="197.7612" width="4" height="2.2388" />
	<rect x="528" y="199.6280" width="4" height="0.3720" />
	<rect x="532" y="199.3485" width="4" height="0.6515" />
	<rect x="536" y="195.3105" width="4" height="4.6895" />
	<rect x="540" y="198.8478" width="4" height="1.1522" />
	<rect x="544" y="196.2062" width="4" height="3.7938" />
	<rect x="548" y="92.8153" width="4" height="107.1847" />
	<rect x="552" y="199.4666" width="4" height="0.5334" />
	<rect x="556" y="199.6113" width="4" height="0.3887" />
	<rect x="560" y="198.4178" width="4" height="1.5822" />
	<rect x="564" y="199.9281" width="4" height="0.0719" />
	<rect x="568" y="199.7053" width="4" height="0.2947" />
	<rect x="572" y="198.9738" width="4" height="1.0262" />
	<rect x="576" y="151.8761" width="4" height="48.1239" />
	<rect x="580" y="199.9788" width="4" height="0.0212" />
	<rect x="584" y="198.8448" width="4" height="1.1552" />
	<rect x="588" y="199.4947" width="4" height="0.5053" />
	<rect x="592" y="199.7930" width="4" height="0.2070" />
	<rect x="596" y="199.6871" width="4" height="0.3129" />
	<rect x="600" y="199.9735" width="4" height="0.0265" />
	<rect x="604" y="199.8861" width="4" height="0.1139" />
	<rect x="608" y="197.7716" width="4" height="2.2284" />
	<rect x="612" y="198.6745" width="4" height="1.3255" />
	<rect x="616" y="191.6168" width="4" height="8.3832" />
	<rect x="620" y="199.7593" width="4" height="0.2407" />
	<rect x="624" y="199.9687" width="4" height="0.0313" />
	<rect x="628" y="199.9760" width="4" height="0.0240" />
	<rect x="632" y="199.9309" width="4" height="0.0691" />
	<rect x="636" y="199.9868" width="4" height="0.0132" />
	<rect x="640" y="166.2811" width="4" height="33.7189" />
	<rect x="644" y="198.9502" width="4" height="1.0498" />
	<rect x="648" y="199.5646" width="4" height="0.4354" />
	<rect x="652" y="199.9840" width="4" height="0.0160" />
	<rect x="656" y="199.9631" width="4" height="0.0369" />
	<rect x="660" y="199.9806" width="4" height="0.0194" />
	<rect x="664" y="199.9839" width="4" height="0.0161" />
	<rect x="668" y="199.8773" width="4" height="0.1227" />
	<rect x="672" y="176.0984" width="4" height="23.9016" />
	<rect x="676" y="199.5566" width="4" height="0.4434" />
	<rect x="680" y="199.9841" width="4" height="0.0159" />
	<rect x="684" y="199.4704" width="4" height="0.5296" />
	<rect x="688" y="199.9775" width="4" height="0.0225" />
	<rect x="692" y="199.8709" width="4" height="0.1291" />
	<rect x="696" y="199.9947" width="4" height="0.0053" />
	<rect x="700" y="199.8150" width="4" height="0.1850" />
	<rect x="704" y="199.4690" width="4" height="0.5310" />
	<rect x="708" y="199.8120" width="4" height="0.1880" />
	<rect x="712" y="199.8069" width="4" height="0.1931" />
	<rect x="716" y="199.7036" width="4" height="0.2964" />
	<rect x="720" y="199.9846" width="4" height="0.0154" />
	<rect x="724" y="199.9824" width="4" height="0.0176" />
	<rect x="728" y="199.7120" width="4" height="0.2880" />
	<rect x="732" y="199.8075" width="4" height="0.1925" />
	<rect x="736" y="199.6418" width="4" height="0.3582" />
	<rect x="740" y="199.9951" width="4" height="0.0049" />
	<rect x="744" y="199.9891" width="4" height="0.0109" />
	<rect x="748" y="199.9776" width="4" height="0.0224" />
	<rect x="752" y="199.8337" width="4" height="0.1663" />
	<rect x="756" y="199.9976" width="4" height="0.0024" />
	<rect x="760" y="199.8928" width="4" height="0.1072" />
	<rect x="764" y="199.9966" width="4" height="0.0034" />
	<rect x="768" y="161.7416" width="4" height="38.2584" />
	<rect x="772" y="199.9936" width="4" height="0.0064" />
	<rect x="776" y="199.9930" width="4" height="0.0070" />
	<rect x="780" y="199.9494" width="4" height="0.0506" />
	<rect x="784" y="199.9538" width="4" height="0.0462" />
	<rect x="788" y="199.9812" width="4" height="0.0188" />
	<rect x="792" y="199.9831" width="4" height="0.0169" />
	<rect x="796" y="199.9839" width="4" height="0.0161" />
	<text x="130" y="220">320</text>
	<text x="258" y="220">640</text>
	<text x="386" y="220">960</text>
	<text x="514" y="220">1280</text>
	<text x="642" y="220">1600</text>
	<text x="770" y="220">1920</text>
	<g fill="red">
	<text x="410" y="240">1024</text>
	<text x="514" y="240">1280</text>
	<text x="550" y="240">1366</text>
	</g>
</svg>

I fully expected something that looked like a bell curve of screen sizes,
especially in the desktop range where people can resize their browser windows
at will. But instead, even at those desktop sizes, there were some screensizes
that vastly outpace other screensizes.

The largest spikes were around 1024px (16%), 1280 (27%), and 1366 (15%), and I
frankly I have no idea why that would be.
