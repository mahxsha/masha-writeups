# Writeup: Boo

The flag for this challenge is divided into four specific parts based on the clues in the description:
`upCTF{MyFirstName_ArchitecturalStyle_Year_Place}`

### Part 1: The Ghost (MyFirstName)
The description is written in the first person: *"I’ve had"*, *"I’m standing"*, *"but I never can"*, *"frames my name"*. 

Because the narrator says they can never leave the place they are describing and they were the "last to get to a stone" where they sleep, it is clear this is a **ghost**. 

To find the location, we look at the physical clues:
* **"Blue tiles full of history"**: Typical of Portuguese architecture.
* **"Sound of iron on iron"**: Reminiscent of a train station.

Searching for "train station with blue tiles" leads to the **São Bento Station** in Porto. Checking Wikipedia for the history of the station reveals:
> *"Convent of São Bento da Avé Maria... The building had been a monastery until it was destroyed by fire in 1783."*

This confirms the location, matching the line: *"since the fire in 1783, this place has never felt the same."*

Searching for "São Bento ghost" reveals the local story: the station could only be built once the **last nun** of the convent died. She was the one who was the "last to get to the stone." Her name was **Maria da Glória Dias Guimarães**.
* **Flag Part 1 (MyFirstName):** `Maria`

### Parts 2, 3, & 4: The Stone (ArchitecturalStyle_Year_Place)
The second half of the challenge asks: *"Can you find where the stone that frames my name now stands, and tell me: who built it, when, and where do I sleep?"*

Searching for where the nuns of the São Bento convent were buried leads to various blogs and historical archives. The nuns remains were moved to an ossuary. One specific source ([nils.mipi.de](https://nils.mipi.de/2016/07/a-love-story-to-lose-your-head/)) shows a clear image of the ossuary.

`CONSTRUCAO MANUELINA PRIMITIVA DE 1525`

This gives us the final three pieces:
* **Flag Part 2 (ArchitecturalStyle):** `Manuelina`
* **Flag Part 3 (Year):** `1525`
* **Flag Part 4 (Place):** `PradoDoRepouso` (Cemitério do Prado do Repouso)

### Final Flag
`upCTF{Maria_Manueline_1525_CemiteryOfPradoDoRepouso}`