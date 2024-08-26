question:
inishing on the same key it started. A musical pyramid must have
an odd number of keys.
For example, [1, 2, 3, 2, 1], [3, 4, 3], [100, 101, 100] and [7] are all musical pyramids.
The following are not musical pyramids:
• [2, 2] and [5, 6, 6, 5] since they do not have an odd number of keys.
• [1, 3, 5, 3, 1] since the keys are not consecutive (ascending or descending).
• [1, 2, 3, 4, 3] since the sequence does not stop ascending and start descending precisely in the
middle. It also does not finish on the same key it started.
What is the minimum number of keys you need to remove from the sequence to form a musical pyramid?

solution:
create two large arrays/list which can contain all numbers (up to 1M ) hence the list size will be 100001.

core idea: if a number's prev number exists in the list, then increase its weight. 

ie. if input is 1 2 3.
when handling first number 1, its weight will be set to 1
when handling second number 2, its weight will be its prev number (in this case 1)'s weight +1 -> 2
when handling 3rd number, its weight will be number 2's weight + 1. --> 3
3 means number 2 and 1 can be found on its left side with the right oder (other numbres are allowed in between 1 and 2. but if 2 comes first than 1, then number 2's weight will be 1 
instead of 2).

once finish the first loop from left to right, we can have a list of weights.
we store the results into a list of N elements.

now we repeat the same process but start from right to left. also store the results into another list.

now we use a final loop.

for number in the first list , the corresponding position in second list will be N-i-1. find their minnium and then choose max from next.

if the same number found from both sides, then we will take the mim weights from them as the min represents the number of common number sequences. we keep going until
we found the largest one.

cr = 0
for i in range(n):
    cr = max(cr, min(bcl[i], bcr[n - 1 - i]))

instead of using a big list, we can use dictionary.

answer = None

# Open the input and output files.
input_file = open("compin.txt", "r")
output_file = open("compout.txt", "w")

# Read the value of N.
n = int(input_file.readline().strip())

# Read the sequence of keys.
input_line = input_file.readline().strip()
pyr = list(map(int, input_line.split()))

# TODO: This is where you should compute your solution. Store the minimum
# number of keys you need to remove from the sequence into the variable answer.

lreq = [0] * 100001
rreq = [0] * 100001
bcl = [0] * n
bcr = [0] * n
    
for i in range(n):
    lreq[pyr[i]] = lreq[pyr[i] - 1] + 1
    bcl[i] = lreq[pyr[i]]
    
for i in range(n):
    rreq[pyr[n - 1 - i]] = rreq[pyr[n - 1 - i] - 1] + 1
    bcr[i] = rreq[pyr[n - 1 - i]]

cr = 0
for i in range(n):
    cr = max(cr, min(bcl[i], bcr[n - 1 - i]))
  
answer = n- (2*cr-1)
print(answer)
# Write the answer to the output file.
output_file.write("%d\n" % (answer))


# Open the input and output files.
input_file = open("compin.txt", "r")
output_file = open("compout.txt", "w")

# Read the value of N.
n = int(input_file.readline().strip())

# Read the sequence of keys.
input_line = input_file.readline().strip()
pyr = list(map(int, input_line.split()))

# TODO: This is where you should compute your solution. Store the minimum
# number of keys you need to remove from the sequence into the variable answer.


bcl = [0] * n
bcr = [0] * n

ld = {}
rd = {}

for i in range(n):
    v = pyr[i]
    prev = ld.get(v-1,0) + 1
    ld[v] = prev
    bcl[i] = prev
    
for i in range(n):
    v = pyr[n - 1 - i]
    prev = rd.get(v-1,0) + 1
    rd[v] = prev
    bcr[i] = prev    

cr = 0
for i in range(n):
    cr = max(cr, min(bcl[i], bcr[n - 1 - i]))
  
answer = n- (2*cr-1)
print(answer)
# Write the answer to the output file.
output_file.write("%d\n" % (answer))

# Finally, close the input/output files.
input_file.close()
output_file.close()


