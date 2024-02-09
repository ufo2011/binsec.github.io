---
layout: post
title:  "ESEC/FSE (2023): research paper"
categories: new publication
paper-title: "Scalable Program Clone Search through Spectral Analysis"
topic: "binary similarity, clone seach, binary analysis"
pdf: "/assets/publications/papers/2023-fse.pdf"
date: 2023-12-04
redirect_from: /new/publication/1970/01/01/nutshell-fse-23.html
---

We consider the problem of program clone search, i.e. given a target program and a repository of known programs (all in executable format), the goal is to find the program in the repository most similar to the target program -- with potential applications in terms of reverse engineering, program clustering, malware lineage and software theft detection. Recent years have witnessed a blooming in code similarity techniques, yet most of them focus on function-level similarity and function clone search, while we are interested in program-level similarity and program clone search. Actually, our study shows that prior similarity approaches are either too slow to handle large program repositories, or not precise enough, or yet not robust against slight variations introduced by compilers, source code versions or light obfuscations. We propose a novel spectral analysis method for program-level similarity and program clone search called Programs Spectral Similarity (PSS). In a nutshell, PSS one-time spectral feature extraction is tailored for large repositories, making it a perfect fit for program clone search. We have compared the different approaches with extensive benchmarks, showing that PSS reaches a sweet spot in terms of precision, speed and robustness.


## Further information
- Read the [paper](/assets/publications/papers/2023-fse.pdf) 
- [Published](https://dl.acm.org/doi/proceedings/10.1145/3611643) in the "Proceedings of the 31st ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering".

