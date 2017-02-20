---
categories: String data structures
...

A trie is a data structure that allows quick searches in a dictionary. Concretely, searching for a word of length $n$ in a trie has a complexity of $O(n)$, and building a trie has a complexity of $O(L)$, where $L$ is the total length of the words in the dictionary.

A trie is a tree rooted at an empty node, and in which each way from the root to a leaf represents a word in the dictionary. In order to accomplish this task, a node at depth $k$ represents a substring of length $k$, that is the substring represented by the direct antecesor of the current node plus an additional character. Of course, the root represents the empty string. If a certain node represents a whole word in the dictionary, an additional flag should be used to mark this fact.

Tries are useful in problems of longest common prefix-suffix-substrings, string matching, string matching with mistakes, etc. Also, there are generalizations of tries (suffix tries-trees), that are useful in other kinds of problems.

Sample code (c++):

~~~{.c++}
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
* [Xor Queries](https://www.codechef.com/problems/XRQRS) [^1]


[^1]: <https://discuss.codechef.com/questions/61310/xrqrs-editorial>
