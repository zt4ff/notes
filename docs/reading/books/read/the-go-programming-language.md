# The GO Programming Language Exercises Solutions

## Chapter 1

### Exercise 1.1 Modif y the echo program to also print os.Args[0], the name of the command that invoked it.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for i, arg := range os.Args {
		if i > 0 {
			fmt.Print(" ")
		}
		fmt.Print(arg)
	}
	fmt.Println()
}
```

### Exercise 1.2 Modify the echo program to print the index and value of each of its arguments, one per line.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for i, arg := range os.Args {
		fmt.Println(i, arg)
	}
}
```

### Exercise 1.3 Exercise 1.3: Experiment to measure the difference in running time between our potentially inefficient versions and the one that uses strings.Join. (Section 1.6 illustrates part of the time package, and Section 11.4 shows how to write benchmark tests for systematic per- formance evaluation.)

```go
package echo

import (
	"os"
	"strings"
	"testing"
)

func echoConcat() string {
	s := ""
	for _, arg := range os.Args[1:] {
		s += arg
	}
	return s
}

func echoJoin() string {
	return strings.Join(os.Args[1:], "")
}

func BenchmarkEchoConcat(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = echoConcat()
	}
}

func BenchmarkEchoJoin(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = echoJoin()
	}
}
```

### Exercise 1.4: Modif y dup2 to print the names of all files in which each duplicated line occurs.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	counts := make(map[string]int)
	files := make(map[string]map[string]bool)

	for _, name := range os.Args[1:] {
		f, err := os.Open(name)
		if err != nil {
			continue
		}
		input := bufio.NewScanner(f)
		for input.Scan() {
			line := input.Text()
			counts[line]++
			if files[line] == nil {
				files[line] = make(map[string]bool)
			}
			files[line][name] = true
		}
		f.Close()
	}

	for line, n := range counts {
		if n > 1 {
			fmt.Print(n, "\t", line)
			for name := range files[line] {
				fmt.Print("\t", name)
			}
			fmt.Println()
		}
	}
}
```

### Exercise 1.5: Change the Lissajous program’s color palette to green on black, for added authenticity. To cre ate the web color #RRGGBB, use color.RGBA{0xRR, 0xGG, 0xBB, 0xff}, where each pair of hexadecimal digits represents the intensity of the red, green, or blue component of the pixel.

```go
package main

import (
	"image"
	"image/color"
	"image/gif"
	"math"
	"math/rand"
	"os"
)

var palette = []color.Color{
	color.Black,
	color.RGBA{255, 0, 0, 255},
	color.RGBA{0, 255, 0, 255},
	color.RGBA{0, 0, 255, 255},
}

const (
	whiteIndex = 0
)

func main() {
	const (
		cycles  = 5
		res     = 0.001
		size    = 100
		nframes = 64
		delay   = 8
	)

	freq := rand.Float64() * 3.0
	anim := gif.GIF{LoopCount: nframes}
	phase := 0.0
	for i := 0; i < nframes; i++ {
		rect := image.Rect(0, 0, 2*size+1, 2*size+1)
		img := image.NewPaletted(rect, palette)
		colorIndex := uint8(i%len(palette-1) + 1)
		for t := 0.0; t < cycles*2*math.Pi; t += res {
			x := math.Sin(t)
			y := math.Sin(t*freq + phase)
			img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), colorIndex)
		}
		phase += 0.1
		anim.Delay = append(anim.Delay, delay)
		anim.Image = append(anim.Image, img)
	}
	gif.EncodeAll(os.Stdout, &anim)
}
```

### Exercise 1.6: Modify the Lissajous program to produce images in multiple colors by adding more values to palette and then displaying them by changing the third argument of Set-ColorIndex in some interesting way.

```go
package main

import (
	"fmt"
	"math"
)

const (
	width, height = 600, 320
	cells         = 100
	xyrange       = 30.0
)

func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' style='stroke: grey; fill: white; stroke-width: 0.7' width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i+1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j+1)
			dx, dy := corner(i+1, j+1)
			z := f((float64(i)/cells-0.5)*2*xyrange, (float64(j)/cells-0.5)*2*xyrange)
			color := "blue"
			if z > 0 {
				color = "red"
			}
			fmt.Printf("<polygon points='%g,%g %g,%g %g,%g %g,%g' style='fill:%s'/>", ax, ay, bx, by, cx, cy, dx, dy, color)
		}
	}
	fmt.Println("</svg>")
}

func corner(i, j int) (float64, float64) {
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)
	z := f(x, y)
	sx := width/2 + (x-y)*math.Cos(math.Pi/6)*width/xyrange/2
	sy := height/2 + (x+y)*math.Sin(math.Pi/6)*width/xyrange/2 - z*height*0.4
	return sx, sy
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y)
	return math.Sin(r) / r
}
```

### Exercise 1.7: The function call io.Copy(dst, src) reads from src and writes to dst. Use it instead of ioutil.ReadAll to copy the response body to os.Stdout without requiring a buffer large enough to hold the entire stream. Be sure to check the error result of io.Copy.

```go
package main

import (
	"io"
	"net/http"
	"os"
)

func main() {
	resp, _ := http.Get(os.Args[1])
	io.Copy(os.Stdout, resp.Body)
	resp.Body.Close()
}
```

### Exercise 1.8: Modify fetch to add the prefix http:// to each argument URL if it is missing. You might want to use strings.HasPrefix.

```go
package main

import (
	"io"
	"net/http"
	"os"
	"strings"
)

func main() {
	url := os.Args[1]
	if !strings.HasPrefix(url, "http://") && !strings.HasPrefix(url, "https://") {
		url = "http://" + url
	}
	resp, _ := http.Get(url)
	io.Copy(os.Stdout, resp.Body)
	resp.Body.Close()
}
```

### Exercise 1.9: Modify fetch to also print the HTTP status code, found in resp.Status.

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
)

func main() {
	resp, _ := http.Get(os.Args[1])
	fmt.Println(resp.Status)
	io.Copy(os.Stdout, resp.Body)
	resp.Body.Close()
}
```

### Exercise 1.10: Find a web site that produces a large amount of data. Investigate caching by running fetchall twice in succession to see whether the reported time changes much. Do you get the same content each time? Modify fetchall to print its output to a file so it can be examined.

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"time"
)

func fetch(url string, ch chan<- string) {
	start := time.Now()
	resp, _ := http.Get(url)
	n, _ := io.Copy(io.Discard, resp.Body)
	resp.Body.Close()
	ch <- fmt.Sprintf("%.2fs %7d %s", time.Since(start).Seconds(), n, url)
}

func main() {
	start := time.Now()
	ch := make(chan string)
	for _, url := range os.Args[1:] {
		go fetch(url, ch)
	}
	for range os.Args[1:] {
		fmt.Println(<-ch)
	}
	fmt.Printf("%.2fs elapsed\n", time.Since(start).Seconds())
}
```

### Exercise 1.11: Try fetchall with longer argument lists, such as samples from the top million web sites available at alexa.com. How does the program behave if a web site just doesn’t respond? (Section 8.9 describes mechanisms for coping in such cases.)

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"time"
)

func fetch(url string, ch chan<- string) {
	start := time.Now()
	resp, err := http.Get(url)
	if err != nil {
		ch <- err.Error()
		return
	}
	n, _ := io.Copy(io.Discard, resp.Body)
	resp.Body.Close()
	ch <- fmt.Sprintf("%.2fs %7d %s", time.Since(start).Seconds(), n, url)
}

func main() {
	ch := make(chan string)
	for _, url := range os.Args[1:] {
		go fetch(url, ch)
	}
	for range os.Args[1:] {
		fmt.Println(<-ch)
	}
}
```

### Exercise 1.2: Modify the Lissajous server to read parameter values from the URL. For example, you mig ht arrange it so that a URL like http://localhost:8000/?cycles=20 sets the number of cycles to 20 instead of the default 5. Use the strconv.Atoi func tion to convert the string parameter into an integer. You can see its documentation wit h go doc strconv.Atoi.

```go
package main

import (
	"image"
	"image/color"
	"image/gif"
	"math"
	"math/rand"
	"net/http"
)

var palette = []color.Color{color.White, color.Black}

func lissajous(w http.ResponseWriter) {
	const (
		cycles  = 5
		res     = 0.001
		size    = 100
		nframes = 64
		delay   = 8
	)
	freq := rand.Float64() * 3.0
	anim := gif.GIF{LoopCount: nframes}
	phase := 0.0
	for i := 0; i < nframes; i++ {
		rect := image.Rect(0, 0, 2*size+1, 2*size+1)
		img := image.NewPaletted(rect, palette)
		for t := 0.0; t < cycles*2*math.Pi; t += res {
			x := math.Sin(t)
			y := math.Sin(t*freq + phase)
			img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), 1)
		}
		phase += 0.1
		anim.Delay = append(anim.Delay, delay)
		anim.Image = append(anim.Image, img)
	}
	gif.EncodeAll(w, &anim)
}

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		lissajous(w)
	})
	http.ListenAndServe(":8000", nil)
}
```

## Chapter 2

### Exercise 2.1: Add types, constants, and functions to tempconv for processing temperatures in the Kelvin scale, where zero Kelvin is −273.15°C and a difference of 1K has the same magnitude as 1°C.

```go
package tempconv

type Kelvin float64

const AbsoluteZeroC Celsius = -273.15

func CToK(c Celsius) Kelvin {
	return Kelvin(c - AbsoluteZeroC)
}

func KToC(k Kelvin) Celsius {
	return Celsius(k) + AbsoluteZeroC
}

func FToK(f Fahrenheit) Kelvin {
	return CToK(FToC(f))
}

func KToF(k Kelvin) Fahrenheit {
	return CToF(KToC(k))
}
```

### Exercise 2.2: Write a general-purpose unit-conversion program analogous to cf that reads numbers from its command-line arguments or from the standard input if there are no arguments, and converts each number into units like temperature in Celsius and Fahren heit, length in feet and meters, weig ht in pounds and kilograms, and the like.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"
)

type Celsius float64
type Fahrenheit float64
type Feet float64
type Meters float64
type Pounds float64
type Kilograms float64

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }

func FtToM(f Feet) Meters       { return Meters(f * 0.3048) }
func MToFt(m Meters) Feet       { return Feet(m / 0.3048) }

func LbToKg(p Pounds) Kilograms { return Kilograms(p * 0.45359237) }
func KgToLb(k Kilograms) Pounds { return Pounds(k / 0.45359237) }

func convert(v float64) {
	c := Celsius(v)
	f := Fahrenheit(v)
	ft := Feet(v)
	m := Meters(v)
	lb := Pounds(v)
	kg := Kilograms(v)

	fmt.Printf("%g°C = %g°F, %g°F = %g°C\n", c, CToF(c), f, FToC(f))
	fmt.Printf("%gft = %gm, %gm = %gft\n", ft, FtToM(ft), m, MToFt(m))
	fmt.Printf("%glb = %gkg, %gkg = %glb\n", lb, LbToKg(lb), kg, KgToLb(kg))
}

func main() {
	if len(os.Args) > 1 {
		for _, arg := range os.Args[1:] {
			v, err := strconv.ParseFloat(arg, 64)
			if err != nil {
				continue
			}
			convert(v)
		}
		return
	}

	in := bufio.NewScanner(os.Stdin)
	for in.Scan() {
		v, err := strconv.ParseFloat(in.Text(), 64)
		if err != nil {
			continue
		}
		convert(v)
	}
}
```

### Exercise 2.3: Rewrite PopCount to use a loop instead of a single expression. Compare the performance of the two versions. (Section 11.4 shows how to compare the per formance of different implementations systematically.)

```go
package popcount

var pc [256]byte

func init() {
	for i := range pc {
		pc[i] = pc[i/2] + byte(i&1)
	}
}

func PopCountExpr(x uint64) int {
	return int(pc[byte(x>>(0*8))] +
		pc[byte(x>>(1*8))] +
		pc[byte(x>>(2*8))] +
		pc[byte(x>>(3*8))] +
		pc[byte(x>>(4*8))] +
		pc[byte(x>>(5*8))] +
		pc[byte(x>>(6*8))] +
		pc[byte(x>>(7*8))])
}

func PopCountLoop(x uint64) int {
	n := 0
	for i := 0; i < 8; i++ {
		n += int(pc[byte(x>>(i*8))])
	}
	return n
}
```

### Exercise 2.4: Write a version of PopCount that counts bits by shifting its argument through 64 bit positions, testing the rightmost bit each time. Compare its performance to the table-lookup version.

```go
package popcount

func PopCountShift(x uint64) int {
	n := 0
	for i := 0; i < 64; i++ {
		if x&1 == 1 {
			n++
		}
		x >>= 1
	}
	return n
}
```

## Exercise 2.5: The expression x&(x-1) clears the rightmost non-zero bit of x. Write a version of PopCount that counts bits by using this fact, and assess its performance.

```go
package popcount

func PopCountClear(x uint64) int {
	n := 0
	for x != 0 {
		x &= x - 1
		n++
	}
	return n
}
```
