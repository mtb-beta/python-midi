<a href="https://travis-ci.org/vishnubob/python-midi"><img src="https://travis-ci.org/vishnubob/python-midi.svg?branch=master" alt='Build Statis' /></a>
<a href='https://coveralls.io/github/vishnubob/python-midi?branch=master'><img src='https://coveralls.io/repos/vishnubob/python-midi/badge.svg?branch=master&service=github' alt='Coverage Status' /></a>

=Python MIDI=

ラッパーとして驚愕の能力を持つPythonですが、MIDIデータを容易に扱う機能は提供されていません。PythonからMIDIを操作することを目標とした異なるパッケージが10種類ほど存在します。しかし、PythonからのMIDIを操作することが網羅的にカバーされたものはありません。

このtoolkitはPythonからMIDIを操作することを完全に達成することを目指しています。その中でも特に、ハードウェアから独立したハイレベルなフレームワークを提供するために努力します。それはMIDIの流れを簡単に操作するためのシーケンスやレコード、プレイバックを合理的な粒度のオブジェクトとして提供することを試みています。上位の時間概念をもつことは大切なことであり、そのイベントフレームワークは例えば、自動的にticksを実際の経過時間に変換した自動的な接続を提供します。

このPython MIDI toolkitは約2年の間に点在した開発の代理です。もしあなたが私のようにPython MIDI フレームワークに時間を費やしてきた何者かであれば、このtoolkitは良いものかもしれません。このtoolkitは完璧ではありませんが、大きな特徴を提供しています。

==特徴==

* 個々のMIDIイベントを代理で扱うハイレベルクラス
* 使用者のMIDIイベントを管理することができるマルチトラックコンテナ
* トラックのなかで能動的にレンポを変更するテンポマップ
* あなたが作成したMIDIトラックを読み込んで書き出すReaderとWriter

==インストール==

[http://docs.python.org/2/install/index.html 通常のPythonモジュールのインストールの手順 ]を参考にしてください。

<pre>
python setup.py install
</pre>

===MIDI Fileの確認===

To examine the contents of a MIDI file run
MIDIファイルの中身を確認する場合は、次のコマンドを実行してください。

<pre>
$ mididump.py mary.mid
</pre>

これは"Mary had a Little Lamb"をPythonとして実行してプリントアウトします。

==使用例==

===編集したMIDI Fileの書き出し===

編集したMIDIトラックを書き出すのは簡単です。

<pre>
import midi
# MIDI パターンのインスタンス生成 (複数のトラックのリストを含みます)
pattern = midi.Pattern()
# MIDI トラックのインスタンス生成 (MIDIイベントのリストを含みます)
track = midi.Track()
# MIDI パターンにMIDIトラックをappend
pattern.append(track)
# MIDI ノートオンイベントのインスタンスを生成し、trackにappend
on = midi.NoteOnEvent(tick=0, velocity=20, pitch=midi.G_3)
track.append(on)
# MIDI ノートオフイベントのインスタンスを生成し、trackにappend
off = midi.NoteOffEvent(tick=100, pitch=midi.G_3)
track.append(off)
# MIDI ノート終了イベントのインスタンスを生成し、trackにappend
eot = midi.EndOfTrackEvent(tick=1)
track.append(eot)
# MIDI パターンを出力
print pattern
# MIDI パターンを書き出し
midi.write_midifile("example.mid", pattern)
</pre>

単一のMIDIファイルは、オブジェクトの階層表現で表されます。その最上位層はPatternです。PatternはTrackのリストを含んでいます。TrackはMIDIイベントのリストを含んでいます。

MIDI Patternクラスは、標準的なPythonのリストを継承しています。そのため、全てのリストの特徴を受けついています。例えば、append()、extend()、スライスやイテレーションなどです。PatternはMIDIファイルのメタデータ合わせて含んでいます。そこには、解像度やMIDIフォーマットが記述されています。

MIDI Trackクラスも、標準的なPythonのリストを継承しています。そのため、Patternのようにメタデータを含んでいません。しかし、Trackの中に含まれる全てのMIDI Eventsを操作する補助機能を備えています。

27の異なるMIDI Eventsをサポートしています。この例では、3つの異なるMIDI Eventsが作成されてMIDI Trackに追加されています。

# NoteOnEventはノートの始まりを表現します。それはピアノの演奏者がピアノのキーを打鍵するようなものです。tickはそのイベントが発生した時間を表します。pitchはそのノートが押されたキーの高さです。そして、velocityはどれだけ強くキーを叩いたかになります。

# NoteOffEventはノートの終わりを表現します。それはちょうどピアノ演奏者が鍵盤から指を離したことにあたります。繰り返しになりますが、tickはこのイベントが発生した時刻であり、pitchは離されたノートにあたります。そして、velocityは現実世界では同概念がなく普通は無視されます。０のvelocityをもつNoteOnEventsはNoteOffEvetsに相当します。

# EndOfTrackEventは特殊なイベントです。それは、MIDIシーケンスがソフトウェアに楽曲の最後を示す為に使われます。マルチトラックを含むPatternsの作成では、あなたはEndOfTrackイベントを楽曲につき１つだけ追加することを必要です。一般的なMIDIソフトウェアは、MIDIファイルがEndOfTrackイベントを含んでいた際にそれの読み込みを拒否しません。

あなたはEndOfTrackEventがtick1の値をもっていることに気付くかもしれません。これはMIDIのtickeが相対時刻で表現されていることが原因にあります。現実的にはMIDITrackEventのtickオフセットは、そのtickと直前までのtickの合計値になります。これは今回の例で言えば、、EndOfTrackEventはtick101で発生することになります。(0 + 100 + 1)

====注釈: MIDI Tickとはなにか====

ticksの問題は、解像度とテンポ以外の情報を知らない限り、何の情報もあなたに与えないということです。このコードは、これらの問題に対して全てのあなたが対応すべきこと、例えばあなたがビートについて心配していた際、ミリセカンドやticksの観点で考えるなど、を扱います。

tickはMIDIトラックの解像度を表現します。テンポは、常に、QPQ(Quarter note per Minute:一分あたりの四分音符) と同じものであるBPM(Beat per Minute:一分あたりのビート）に似ています。この解像度は、PPQ(Pulse per Quater note:四分音符辺りの拍子)としても知られています。TPM(Tick per Beat)とも類似しています。

テンポは２つの要素があります。まず、保存されたMIDIファイルは初期の分解能とテンポを保持しています。あなたはその値をシーケンサーのタイマーを初期化するために利用します。分解能はシーケンサーだけでなく、トラックにも静的な値として考慮されるべきです。MIDIプレイバックの中では、MIDIファイルはシーケンス的に（つまり演奏中に）テンポチェンジイベントを保持しているかもしれません。このイベントは、テンポを指定した時刻に変更します。しかしながら分解能はプレイバック中に初期値から変更されることはありません。

その状態で、MIDIはミリセカンドの中でテンポは表現されます。別の言葉で言うと、あなたはテンポをMirosecods per Beatとして変換していることになります。例えばテンポがBPM120であった場合、Pythonコードは次のように考えます。

<pre>
>>> 60 * 1000000 / 120
500000
</pre>

これは、テンポが500,000 Microsecond per beatであることを示しています。解像度の組み合わせの中で、これはtickを時間に変換することができるでしょう。500,000 Microseconds per beatがあり、分解能が1,000であった場合、1tickはどれくらいの時間でしょうか。

This says the Tempo is 500,000 microseconds per beat.  This, in combination
with the Resolution, will allow you to convert ticks to time.  If there are
500,000 microseconds per beat, and if the Resolution is 1,000 than one tick is
how much time?

<pre>
>>> 500000 / 1000
500
>>> 500 / 1000000.0
0.00050000000000000001
</pre>

別の言葉で言うと、1tickは、0.0005秒、もしくはミリセカンドの半分であることを表しています。分解能が増加とこの数字が小さくなり、逆が増加すると分解能は小さくなります。テンポも同様です。

MIDIは時間署名を符号化していますが、それはテンポに影響を与えていません。しかしながら、ここで時間署名の復習資料を記載しておきます。

http://en.wikipedia.org/wiki/Time_signature

===ディスクからトラックを読み込む方法===

It's just as easy to load your MIDI file from disk.
ディスクからあなたのMIDIファイルを読み込むのはいたって簡単です。

<pre>
import midi
pattern = midi.read_midifile("example.mid")
print pattern
</pre>

==シーケンサー==

あなたはこのtoolkitをLinux上で利用している場合、ALSA(Advanced Linux Sound Architecture)シーケンサーの拡張を利用することができます。それはSWINGラッパーを含んでおり、ALSAを隠蔽したハイレベルシーケンサーインターフェスを提供しています。このシーケンサーはハイレベルイベントフレームワークを理解し、それらのイベントをALSAにアクセス可能な形式に変換するでしょう。これはあなたのハードワークをできるだけ可能にするように試みており、その中にはプレイバック中にテンポチェンジへ対応するが含まれています。あなたはMIDIイベントを次の呼び出しと共に記録することが可能です。ALSAシーケンサーは、OSのハードウェア割り込みをトリガーイベントとして、その瞬間をあなたのMIDIトラックにタイムスタンプします。その時間は、あなたが管理に利用しているのがPythonであるにもかかわらず、とても正確です。

私は、OS-XやWin32シーケンサーでも同様のことをサポートすることに強い興味がありますが、それを手伝ってくれるプラットフォームのエキスパートが必要です、あなたはそのエキスパートですか？あなたが私を手伝ってくれるなら私に連絡をください。


===Scripts for Sequencer===

To examine the hardware and software MIDI devices attached to your
system, run the mididumphw.py script.


<pre>
$ mididumphw.py
] client(20) "OP-1 Midi Device"
]   port(0) [r, w, sender, receiver] "OP-1 Midi Device MIDI 1"
] client(129) "__sequencer__"
] client(14) "Midi Through"
]   port(0) [r, w, sender, receiver] "Midi Through Port-0"
] client(0) "System"
]   port(1) [r, sender] "Announce"
]   port(0) [r, w, sender] "Timer"
] client(128) "FLUID Synth (6438)"
]   port(0) [w, receiver] "Synth input port (6438:0)"
</pre>

In the case shown, [http://qsynth.sourceforge.net/qsynth-index.html qsynth]
is running (client 128), and a hardware synthesizer is attached via
USB (client 20).

To play the example MIDI file, run the midiplay.py script.

<pre>
midiplay.py 128 0 mary.mid
</pre>

==Website, support, bug tracking, development etc.==

You can find the latest code on the home page:
https://github.com/vishnubob/python-midi/

You can also check for known issues and submit new ones to the
tracker: https://github.com/vishnubob/python-midi/issues/

==Thanks==

I originally wrote this to drive the electro-mechanical instruments of Ensemble
Robot, which is a Boston based group of artists, programmers, and engineers.
This API, however, has applications beyond controlling this equipment.  For
more information about Ensemble Robot, please visit:

http://www.ensemblerobot.org/
