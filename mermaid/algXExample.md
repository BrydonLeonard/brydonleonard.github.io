<!-- Mermaid doesn't work on GitHub pages AFAICT, so just pre-rendering this and dumping it here for now -->

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontFamily': 'monospace', 'primaryColor': '#fdfdfd', 'fontSize': '13px' }}}%%
flowchart TD
  start["-  1 2 3 4 5 6 7\nA 1 0 0 1 0 0 1\nB 1 0 0 1 0 0 0\nC 0 0 0 1 1 0 1\nD 0 0 1 0 1 1 0\nE 0 1 1 0 0 1 1\nF 0 1 0 0 0 0 1"]
    start --1--> 1_1

  subgraph frame 1
    1_1["-  <b style='color:limegreen'>1</b> 2 3 4 5 6 7\nA <b style='color:limegreen'>1</b> 0 0 1 0 0 1\nB <b style='color:limegreen'>1</b> 0 0 1 0 0 0\nC 0 0 0 1 1 0 1\nD 0 0 1 0 1 1 0\nE 0 1 1 0 0 1 1\nF 0 1 0 0 0 0 1"]

    1_2["-  <b style='color:red'>1</b> 2 3 <b style='color:red'>1</b> 5 6 <b style='color:red'>1</b>\n<b style='color:limegreen'>A</b> <b style='color:limegreen'>1</b> 0 0 <b style='color:limegreen'>1</b> 0 0 <b style='color:limegreen'>1</b>\nB <b style='color:red'>1</b> 0 0 <b style='color:red'>1</b> 0 0 0\nC 0 0 0 <b style='color:red'>1</b> 1 0 <b style='color:red'>1</b>\nD 0 0 1 0 1 1 0\nE 0 1 1 0 0 1 <b style='color:red'>1</b>\nF 0 1 0 0 0 0 <b style='color:red'>1</b>"]

    1_1 --2--> 1_2
      1_2 --3--> 1_3

    subgraph frame 2-1
      1_3["- <b style='color:limegreen'>2</b> 3 5 6\nD 0 1 1 1"]
      style 1_3 fill:#ff8888
    end

    1_3 --4--> 1_2
    1_2 --5--> 1_1

    1_1 --6--> 2_1

    2_1["-  <b style='color:red'>1</b> 2 3 <b style='color:red'>1</b> 5 6 1\nA <b style='color:red'>1</b> 0 0 <b style='color:red'>1</b> 0 0 1\n<b style='color:limegreen'>B</b> <b style='color:limegreen'>1</b> 0 0 <b style='color:limegreen'>1</b> 0 0 0\nC 0 0 0 <b style='color:red'>1</b> 1 0 1\nD 0 0 1 0 1 1 0\nE 0 1 1 0 0 1 1\nF 0 1 0 0 0 0 1"]


    2_1 --7--> 2_2
    
    subgraph frame 2-2
      2_2["- 2 3 <b style='color:limegreen'>5</b> 6 7\nD 0 1 <b style='color:limegreen'>1</b> 1 0\nE 1 1 0 1 1\nF 1 0 0 0 1\n"]
      
      2_3["- 2 <b style='color:red'>3</b> <b style='color:red'>5</b> <b style='color:red'>6</b> 7\n<b style='color:limegreen'>D</b> 0 <b style='color:limegreen'>1</b> <b style='color:limegreen'>1</b> <b style='color:limegreen'>1</b> 0\nE 1 <b style='color:red'>1</b> 0 <b style='color:red'>1</b> 1\nF 1 0 0 0 1\n"]

      2_2 --8--> 2_3
      2_3 --9--> 2_4

      subgraph frame 2-2-1
        2_4["- <b style='color:limegreen'>2</b> 7\nF <b style='color:limegreen'>1</b> 1"]
        
        2_5["- <b style='color:red'>2</b> <b style='color:red'>7</b>\n<b style='color:limegreen'>F</b> <b style='color:red'>1</b> <b style='color:red'>1</b>"]

        2_4 --10--> 2_5
        2_5 --11--> 2_6

        subgraph frame 2-2-1-1
          2_6["-"]
          style 2_6 fill:limegreen
        end
      end
    end
  end
```