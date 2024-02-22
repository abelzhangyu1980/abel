--a method to calculate hashcode for a given string. a single bit will make a huge difference..

unsigned long oaat(char *key, unsigned long len, unsigned long bits) {
   unsigned long hash, i;
   for (hash = 0, i = 0; i < len; i++) {
       hash += key[i];
       hash += (hash << 10);
       hash ^= (hash >> 6);
    }
   hash += (hash << 3);
   hash ^= (hash >> 11);
   hash += (hash << 15);
   return hash & hashmask(bits);
}

How does oaat work? Inside the for loop, it starts by adding the current byte of the key. That part is similar to what we did when adding up the numbers in a snowflake (Listing 1-10). Those left shifts and exclusive ors are in there to put the key through a blender. Hash functions do this blending to implement an avalanche effect, which means that a small change in the key’s bits makes a huge change to the key’s hashcode. Unless you intentionally created pathological data for this hash function or inserted a huge number of keys, it would be unlikely that you’d get many collisions. This highlights an important point: with a single hash function, there is always a collection of data that will lead to collisions galore and subsequently horrible performance. A fancy hash function like oaat can’t protect against that. Unless we’re concerned about malicious input, though, we can often get away with using a reasonably good hash function and can assume that our hash function will spread the data around.


