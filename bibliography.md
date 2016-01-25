% Bibliografia

Esta es una lista de lectura de material relevante a Rust. Incluye investigación previa que ha - en un momento u otro - influenciado el diseño de Rust, así como publicaciones relacionadas directamente con Rust.

### Sistema de tipos

* [Region based memory management in Cyclone](http://209.68.42.137/ucsd-pages/Courses/cse227.w03/handouts/cyclone-regions.pdf)
* [Safe manual memory management in Cyclone](http://www.cs.umd.edu/projects/PL/cyclone/scp.pdf)
* [Typeclasses: making ad-hoc polymorphism less ad hoc](http://www.ps.uni-sb.de/courses/typen-ws99/class.ps.gz)
* [Macros that work together](https://www.cs.utah.edu/plt/publications/jfp12-draft-fcdf.pdf)
* [Traits: composable units of behavior](http://scg.unibe.ch/archive/papers/Scha03aTraits.pdf)
* [Alias burying](http://www.cs.uwm.edu/faculty/boyland/papers/unique-preprint.ps) - Intentamos algo similar, luego lo abandonamos
* [External uniqueness is unique enough](http://www.cs.uu.nl/research/techreps/UU-CS-2002-048.html)
* [Uniqueness and Reference Immutability for Safe Parallelism](https://research.microsoft.com/pubs/170528/msr-tr-2012-79.pdf)
* [Region Based Memory Management](http://www.cs.ucla.edu/~palsberg/tba/papers/tofte-talpin-iandc97.pdf)

### Concurrencia

* [Singularity: rethinking the software stack](https://research.microsoft.com/pubs/69431/osr2007_rethinkingsoftwarestack.pdf)
* [Language support for fast and reliable message passing in singularity OS](https://research.microsoft.com/pubs/67482/singsharp.pdf)
* [Scheduling multithreaded computations by work stealing](http://supertech.csail.mit.edu/papers/steal.pdf)
* [Thread scheduling for multiprogramming multiprocessors](http://www.eecis.udel.edu/%7Ecavazos/cisc879-spring2008/papers/arora98thread.pdf)
* [The data locality of work stealing](http://www.aladdin.cs.cmu.edu/papers/pdfs/y2000/locality_spaa00.pdf)
* [Dynamic circular work stealing deque](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.170.1097&rep=rep1&type=pdf) - El deque Chase/Lev
* [Work-first and help-first scheduling policies for async-finish task parallelism](http://www.cs.rice.edu/%7Eyguo/pubs/PID824943.pdf) - Mas general que robo de trabajo completamente estricto
* [A Java fork/join calamity](http://www.coopsoft.com/ar/CalamityArticle.html) - critica de la biblioteca fork/join de Java, particularmente su aplicación de robo de trabajo a computación no estricta
* [Scheduling techniques for concurrent systems](http://www.stanford.edu/~ouster/cgi-bin/papers/coscheduling.pdf)
* [Contention aware scheduling](http://www.blagodurov.net/files/a8-blagodurov.pdf)
* [Balanced work stealing for time-sharing multicores](http://www.cse.ohio-state.edu/hpcs/WWW/HTML/publications/papers/TR-12-1.pdf)
* [Three layer cake for shared-memory programming](http://dl.acm.org/citation.cfm?id=1953616&dl=ACM&coll=DL&CFID=524387192&CFTOKEN=44362705)
* [Non-blocking steal-half work queues](http://www.cs.bgu.ac.il/%7Ehendlerd/papers/p280-hendler.pdf)
* [Reagents: expressing and composing fine-grained concurrency](http://www.mpi-sws.org/~turon/reagents.pdf)
* [Algorithms for scalable synchronization of shared-memory multiprocessors](https://www.cs.rochester.edu/u/scott/papers/1991_TOCS_synch.pdf)
* [Epoch-based reclamation](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf).

### Otros

* [Crash-only software](https://www.usenix.org/legacy/events/hotos03/tech/full_papers/candea/candea.pdf)
* [Composing High-Performance Memory Allocators](http://people.cs.umass.edu/~emery/pubs/berger-pldi2001.pdf)
* [Reconsidering Custom Memory Allocation](http://people.cs.umass.edu/~emery/pubs/berger-oopsla2002.pdf)

### Papers *acerca de* Rust

* [GPU Programming in Rust: Implementing High Level Abstractions in a
Systems Level
Language](http://www.cs.indiana.edu/~eholk/papers/hips2013.pdf). Trabajo temprano en GPU por Eric Holk.
* [Parallel closures: a new twist on an old
  idea](https://www.usenix.org/conference/hotpar12/parallel-closures-new-twist-old-idea)
  - no exactamente acerca de Rust, pero por nmatsakis
* [Patina: A Formalization of the Rust Programming
  Language](ftp://ftp.cs.washington.edu/tr/2015/03/UW-CSE-15-03-02.pdf). Formalización temprana de un subconjunto del sistema de tipos, por Eric Reed.
* [Experience Report: Developing the Servo Web Browser Engine using
  Rust](http://arxiv.org/abs/1505.07383). Por Lars Bergstrom.
* [Implementing a Generic Radix Trie in
  Rust](https://michaelsproul.github.io/rust_radix_paper/rust-radix-sproul.pdf). Paper pre-grado por Michael Sproul.
* [Reenix: Implementing a Unix-Like Operating System in
  Rust](http://scialex.github.io/reenix.pdf). Paper pre-grado por  Alex
  Light.
* [Evaluation of performance and productivity metrics of potential
  programming languages in the HPC environment]
  (http://octarineparrot.com/assets/mrfloya-thesis-ba.pdf).
  Tesis pre-grado por lorian Wilkens. Compara C, Go y Rust.
* [Nom, a byte oriented, streaming, zero copy, parser combinators library
  in Rust](http://spw15.langsec.org/papers/couprie-nom.pdf). Por
  Geoffroy Couprie, investigación para VLC.
* [Graph-Based Higher-Order Intermediate
  Representation](http://compilers.cs.uni-saarland.de/papers/lkh15_cgo.pdf). Una RI implementada en impala, un lenguaje similar a Rust.
* [Code Refinement of Stencil
  Codes](http://compilers.cs.uni-saarland.de/papers/ppl14_web.pdf). Otro paper usando Impala.
* [Parallelization in Rust with fork-join and
  friends](http://publications.lib.chalmers.se/records/fulltext/219016/219016.pdf). Tesis de master de Linus
  Farnstrand's.
* [Session Types for
  Rust](http://munksgaard.me/papers/laumann-munksgaard-larsen.pdf). Tesis de master de Philip
  Munksgaard's. Investigacion para Servo.
* [Ownership is Theft: Experiences Building an Embedded OS in Rust - Amit Levy, et. al.](http://amitlevy.com/papers/tock-plos2015.pdf)
* [You can't spell trust without Rust](https://raw.githubusercontent.com/Gankro/thesis/master/thesis.pdf). Tesis de master de Alexis Beingessner.
