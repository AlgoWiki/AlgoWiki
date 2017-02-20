---
categories: String data structures
...

A trie is a data structure that allows quick searches in a dictionary.
Concretely, searching for a word of length $n$ in a trie has a complexity of
$O(n)$, and building a trie has a complexity of $O(L)$, where $L$ is the total
length of the words in the dictionary.

A trie is a tree rooted at an empty node, and in which each way from the root
to a leaf represents a word in the dictionary. In order to accomplish this
task, a node at depth $k$ represents a substring of length $k$, that is the
substring represented by the direct ancestor of the current node plus an
additional character. Of course, the root represents the empty string. If a
certain node represents a whole word in the dictionary, an additional flag
should be used to mark this fact.

## Applications

Tries can be used to compute [longest common prefix](Longest common prefix),
[longest common suffix](Longest common suffix), [longest common
substring](Longest common substring), [string matching](String matching), and
[string matching with mistakes](String matching#Variants), to name a few applications.

By [augmenting](Data structure augmentation) tries, it can be made into a very
flexible data structure. See problem [Xor Queries](#Problems) for an example.

## Variants

When all [suffixes](String#Definitions) of a string are inserted into a trie, the
trie is called a [suffix trie](Suffix trie). This has time and memory
complexity $O(n^2)$, where $n$ is the length of the string. [Suffix
trees](Suffix tree) achieve this in $O(n)$ time and memory by not explicitly
storing all nodes of the trie.

## Implementation in C++

~~~{.cpp}
class Trie{
    struct node{
        node *next[26];   // Assuming alphabet 'a'-'z'
        bool end;

        node(){
            for(int i=0;i<26;i++)next[i]=this;
            end=0;
        }
    };

    node *begin;

public:

    Trie(){
        begin=new node();
    }

    void insert(const string &s){
        int n=s.length();
        node *cur=begin;

        for(int i=0;i<n;i++){
            if(cur->next[s[i]-'a']!=cur)
                cur=cur->next[s[i]-'a'];
            else{
                node *aux=new node();
                cur->next[s[i]-'a']=aux;
                cur=aux;
            }
        }

        cur->end=true;
    }


    bool find(string &s){    // returns true if s is in the input dictionary.
        int n=s.length();
        node *cur=begin;

        for(int i=0;i<n;i++){
            if(cur->next[s[i]-'a']!=cur)
                cur=cur->next[s[i]-'a'];
            else return false;
        }

        return cur->end;
    }
};
~~~


## Problems
- [Xor Queries](https://www.codechef.com/problems/XRQRS) [^1]
- [Prefixes and Suffixes](http://codeforces.com/problemset/problem/432/D)
- [Watto and Mechanism](http://codeforces.com/problemset/problem/514/C)

## External links
- [Tutorial on Trie and example problems](https://threads-iiith.quora.com/Tutorial-on-Trie-and-example-problems)

[^1]: <https://discuss.codechef.com/questions/61310/xrqrs-editorial>