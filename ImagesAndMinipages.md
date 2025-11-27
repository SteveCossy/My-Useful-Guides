It sounds like you're encountering an issue where your minipages are overlapping the first `\includegraphics` command in your LaTeX document. This often happens because the minipages are placed without sufficient vertical spacing or are positioned absolutely, causing them to disregard the space taken by the image.

To fix this, you generally want to ensure that the content is placed sequentially or that the minipages are properly aligned relative to the image. Here are a few common approaches:

**1. Place Minipages After the Image on a New Line:**

The simplest solution is to ensure the minipages start *after* the initial image has completed its rendering and taken up its space. You can achieve this by adding a line break (`\\` or `\par`) after the first `\includegraphics` and before your `minipage` environment.

```latex
\includegraphics[page=1, width=1\linewidth, height=15cm, keepaspectratio]
{/home/stevecos/Documents/images/2025-11-27_Final-DODAGs.png}
\
\begin{minipage}[t]{0.3\textwidth}
  \includegraphics[page=1, width=1\linewidth, height=6cm, keepaspectratio]
      {/home/stevecos/Documents/images/2025-11-27_DODAG-30_05-767.png}
\end{minipage}\hfill
\begin{minipage}[t]{0.66\textwidth} \raggedright
\vspace{-30mm}
 An initial \verb|DIO| started the routing process, triggering the node to create a DODDAG instance 30,
but the \verb|DAG| field was not readable, so no neighbour relationship was established.\\
\verb| 678:00:00:03.538 Node:2 :[INFO: RPL  ] Incoming DIO (id, ver, rank) = (30,240,128) from:fe80::207:7:7:7| \\
\dots
\verb| 705:00:00:03.538 Node:2 :[INFO: IPv6 Nbr  ] Adding neighbor: fe80::207:7:7:7| \\
\dots
\verb| 721:00:00:03.538 Node:2 :[WARN: RPL  ] RPL: Instance or DAG not ready for printing routing table (ID: 30)| \\
A short time later the \verb|DAG| rank is still \verb|640| \& there are still no neighbours:
\begin{verbatim}
1777:00:00:05.770 Node:2 :[INFO: RPL       ] --- RPL Neighbour Set for Instance ID: 30 ---
1778:00:00:05.770 Node:2 :[INFO: RPL       ] RPL: MOP 1, OCP 0, Rank 640, DIOint 12
1779:00:00:05.770 Node:2 :[INFO: RPL       ] --- End of Table for Instance ID: 30 ---
\end{verbatim}
\vspace{10mm}
\end{minipage} \\
```

**2. Using a `figure` or `wrapfig` Environment:**

If you want the text to flow around the image, you might consider using a `figure` environment for the image and then placing your minipages. Or, for more precise wrapping, the `wrapfig` package can be very useful.

**Example with `figure` (if you want the image to be a float):**

```latex
\begin{figure}[h!]
  \centering
  \includegraphics[page=1, width=1\linewidth, height=15cm, keepaspectratio]
  {/home/stevecos/Documents/images/2025-11-27_Final-DODAGs.png}
  \caption{Your first image}
  \label{fig:firstimage}
\end{figure}

% Now your minipages will appear after the figure
\begin{minipage}[t]{0.3\textwidth}
  \includegraphics[page=1, width=1\linewidth, height=6cm, keepaspectratio]
      {/home/stevecos/Documents/images/2025-11-27_DODAG-30_05-767.png}
\end{minipage}\hfill
\begin{minipage}[t]{0.66\textwidth} \raggedright
\vspace{-30mm}
 An initial \verb|DIO| started the routing process, triggering the node to create a DODDAG instance 30,
but the \verb|DAG| field was not readable, so no neighbour relationship was established.\\
\verb| 678:00:00:03.538 Node:2 :[INFO: RPL  ] Incoming DIO (id, ver, rank) = (30,240,128) from:fe80::207:7:7:7| \\
\dots
\verb| 705:00:00:03.538 Node:2 :[INFO: IPv6 Nbr  ] Adding neighbor: fe80::207:7:7:7| \\
\dots
\verb| 721:00:00:03.538 Node:2 :[WARN: RPL  ] RPL: Instance or DAG not ready for printing routing table (ID: 30)| \\
A short time later the \verb|DAG| rank is still \verb|640| \& there are still no neighbours:
\begin{verbatim}
1777:00:00:05.770 Node:2 :[INFO: RPL       ] --- RPL Neighbour Set for Instance ID: 30 ---
1778:00:00:05.770 Node:2 :[INFO: RPL       ] RPL: MOP 1, OCP 0, Rank 640, DIOint 12
1779:00:00:05:770 Node:2 :[INFO: RPL       ] --- End of Table for Instance ID: 30 ---
\end{verbatim}
\vspace{10mm}
\end{minipage} \\
```

**3. Adjusting `\vspace{-30mm}`:**

The `\vspace{-30mm}` in your second minipage is pulling its content upwards by 3cm. If you intend for the minipages to be *below* the main image, this negative vertical space could be a major contributor to the overlapping. Try removing or reducing this negative space, or even adding positive space if needed.

**Consider this scenario visually:**

Imagine you have a large image taking up the top half of a page.
