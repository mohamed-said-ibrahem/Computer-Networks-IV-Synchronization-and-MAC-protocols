# Computer-Networks-IV-Synchronization-and-MAC-protocols
## Full Problem Statement.

The main Project topic is: MAC (medium access control) protocols that determine when devices get to transmit (these protocols *control* *access* to the *medium*). For all but the simplest MAC protocols, the participating devices need to be synchronized in time. The way this is done in practice is to have one node transmit a signal (called a "preamble" or "beacon") and the other nodes will listen for this signal. Since wireless signals travel at the speed of light, all nodes can agree that they will hear the preamble at nearly exactly the same time.

First, we will explore how nodes can listen for a specific preamble to achieve time synchronization. Then, we will wrap up with a Monte Carlo simulation of the capacities of the slotted-ALOHA MAC protocol.

# Part 1: Correlation and Preambles
A preamble is a waveform sent at the beginning of a packet that is used to indicate the start of a packet. The preamble waveform is agreed to ahead-of-time, so the signal carries no data. Receivers will listen for the preamble, and when it's detected the receivers will start to demodulate the rest of the packet. Preambles can also be used to synchronize multiple clients for MAC protocols that require synchronization.
In this part, we will explore how to detect and synchronize to a preamble. Both detection and synchronization can be done using a **correlation**. A correlation of two discrete signals $x$ and $y$ is an infinite-length discrete signal which is defined as
$$ (x \star y)[k] = \sum_{n = -\infty}^{\infty}x^*[n] \cdot y[n + k]$$
In other words, at each index $k$, you take conjugate of the first signal $x$ and offset the second signal $y$ by $k$ indices and then sum the resulting two signals. Intuitively, the correlation measures the similarity of two signals for every offset of the one signal relative to the other. For signals with finite length, the correlation is zero for sufficiently large $|k|$, so often we'll consider the correlation to be finite as well.
### Problem 1.1: Autocorrelation (5 pts)
The autocorrelation of a signal $x$ is the correlation of $x$ with itself. For a uniform random signal $x$ think about what the autocorrelation would look like.
- **Plot the signal and its autocorrelation. Use `numpy.correlate` with `mode='full'`, and plot the absolute value of the autocorrelation.**

So how do we use the correlation to detect a preamble? Let $x$ be the preamble signal, and let $y$ be the signal that the receiver receives. For now, we'll let $y$ be finite length. If the preamble is not received, then we can model $y$ as a random signal, and the correlation between $x$ and $y$ will also be random. However, if the receiver did receive the preamble in the signal $y$, then we can model $y$ as $x$ with some offset plus noise. The correlation between $x$ and $y$ will then have a large amplitude at the time in $y$ when the preamble started. So if we look for large peaks in the correlation we can detect and then also synchronize to a given preamble.

### Problem 1.2: Preamble detection using correlation (10 pts)
Implement preamble detection and synchronization using the correlation.
- **Fill in the `detect_preamble` function below. The function should take two signals and return `None` if the preamble is not found, otherwise it should return the index where the preamble starts.

## Handling Frequency Offsets
In practice, radios will disagree on their center frequencies. This is because a radio's center frequency is determined by a local oscillator (which is fed into the "mixer"), and these oscillators have some error. What this means is that a signal that's transmitted by one radio will be received with a "frequency offset" by another radio.
Unfortunately, if the receiving radio doesn't know what the frequency offset is, then this can cause the correlation to fail. Consider the demonstration below:

As you can see, the correlation with a frequency offset doesn't have a large peak anywhere. This is because the frequency offset causes the received signal to "look" significantly different from the transmitted signal, and so the correlation fails to detect the preamble.
The way this is handled in practice is by using a method called "Schmidl-Cox" synchronization. The intuition is that the frequency offset "corrupts" our signal such that we can't detect it using correlation, but if we were to send two copies of the preamble back-to-back, then we would receive two copies of the corrupted preamble back-to-back. Then, instead of searching for a specific signal, we can search for any signal that repeats in time.
Let $L$ be the length of the repeated segment. Then the preamble has length $2L$. At each index $d$ of the signal $s$, the Schmidl-Cox algorithm calculates the following two values:
$$ P(d) = \sum_{m=0}^{L - 1} s_{d + m}^* s_{d + m + L} $$
$$ R(d) = \frac{1}{2} \sum_{m=0}^{2L - 1} \left| s_{d + m} \right|^2 $$
Then the metric calculated at each index is
$$ M(d) = \frac{\left| P(d) \right|^2}{R(d)^2} $$
Intuitively, $P$ is a correlation between the next $L$ samples and the following $L$ samples and $R$ is a normalization factor. Once the value of $M$ increases beyond a threshold, the preamble is detected. Note that $P$ also has a recursive form:
$$ P(d + 1) = P(d) + (s_{d + L}^* s_{d + 2L}) - (s_{d}^* s_{d + L}) $$

**Food for thought**: How could we use a Schmidl-Cox preamble to estimate the frequency offset? This may not be immediately obvious, but we will do this in Lab 4 to estimate and then correct frequency offsets for when we're receiving data.
### Problem 1.3 Schmidl-Cox Preamble Detection (10 pts)
- **Implement the Schmidl-Cox preamble detector below.**
You can use either the iterative or recursive formulations for calculating $P$. Return the index $d^*$ that maximizes $M(d)$ if $M(d^*)$ is greater than the threshold, otherwise return None.

## Part 2: Slotted-ALOHA Capacity
In the idealized slotted aloha model, time is splitted into multiple slots and the packet transmission time is one full slot. All nodes are perfectly syncronized, and transmit at the beginning of each slot. The transmission probability for each user for each slot is some value $p$.

For each time slot, three cases may happen. 1): There are more than one nodes transmits, thus resulting in a conflict slot. The receiver cannot receive them correctly. 2): There is only one node transmission and thus the receiver can receive it correctly. 3): No node transmits at the current slot.

In this section, we will verify the slotted-ALOHA capacity curve through Monte-Carlo simulation.

### Problem 2.1 Slotted ALOHA Simulation (10 pts)
- **Fill in the function below.**
The function will take a number of users, a number of time slots, and a probability $p$ that a given user will transmit on a given timeslot. The function will return the ratio of successful timeslots (timeslots in which a successful transmission occurred) to total number of timeslots.

### Problem 2.2 Verifying Slotted ALOHA capacity (5 pts)
Run your simulation over 1000 values of $p$ between 0 and 1 with 10 users and 10k time slots.
- **Plot the timeslot success ratio as a function of $p$ **
Verify that the resulting plot matches the theoretical curve.


