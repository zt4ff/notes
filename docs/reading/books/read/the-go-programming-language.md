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

## Chaper 3

### Exercise 3.1: If the function f returns a non-finite float64 value, the SVG file will contain invalid <polygon> elements (although many SVG renderers handle this gracefully). Modify the program to skip invalid polygons.

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
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay, ok1 := corner(i+1, j)
			bx, by, ok2 := corner(i, j)
			cx, cy, ok3 := corner(i, j+1)
			dx, dy, ok4 := corner(i+1, j+1)
			if !(ok1 && ok2 && ok3 && ok4) {
				continue
			}
			fmt.Printf("<polygon points='%g,%g %g,%g %g,%g %g,%g'/>", ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Println("</svg>")
}

func corner(i, j int) (float64, float64, bool) {
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)
	z := f(x, y)
	if math.IsNaN(z) || math.IsInf(z, 0) {
		return 0, 0, false
	}
	sx := width/2 + (x-y)*math.Cos(math.Pi/6)*width/xyrange/2
	sy := height/2 + (x+y)*math.Sin(math.Pi/6)*width/xyrange/2 - z*height*0.4
	if math.IsNaN(sx) || math.IsNaN(sy) || math.IsInf(sx, 0) || math.IsInf(sy, 0) {
		return 0, 0, false
	}
	return sx, sy, true
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y)
	return math.Sin(r) / r
}
```

### Exercise 3.2: Experiment with visualizations of other functions fro m the math package. Can you produce an egg box, moguls, or a saddle?

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
	xyscale       = width / 2 / xyrange
	zscale        = height * 0.4
	angle         = math.Pi / 6
)

var sin30, cos30 = math.Sin(angle), math.Cos(angle)

func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
		"style='stroke: grey; fill: white; stroke-width: 0.7' "+
		"width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay, ok1 := corner(i+1, j)
			bx, by, ok2 := corner(i, j)
			cx, cy, ok3 := corner(i, j+1)
			dx, dy, ok4 := corner(i+1, j+1)

			if !ok1 || !ok2 || !ok3 || !ok4 {
				continue
			}

			fmt.Printf("<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",
				ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Println("</svg>")
}

func corner(i, j int) (float64, float64, bool) {
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)

	z := f(x, y)

	if !math.IsFinite(z) {
		return 0, 0, false
	}

	sx := width/2 + (x-y)*cos30*xyscale
	sy := height/2 + (x+y)*sin30*xyscale - z*zscale

	if !math.IsFinite(sx) || !math.IsFinite(sy) {
		return 0, 0, false
	}

	return sx, sy, true
}

func eggBox(x, y float64) float64 {
	return math.Sin(x) + math.Sin(y)
}

func moguls(x, y float64) float64 {
	return math.Sin(x) * math.Cos(y) * 10
}

func saddle(x, y float64) float64 {
	return (x*x - y*y) / 300
}

// Use one of the above functions
func f(x, y float64) float64 {
	return saddle(x, y)
}
```

### Exercise 3.3: Color each polygon based on its height, so that the peaks are colored red(#ff0000) and the val leys blue (#0000ff).

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
	xyscale       = width / 2 / xyrange
	zscale        = height * 0.4
	angle         = math.Pi / 6
)

var sin30, cos30 = math.Sin(angle), math.Cos(angle)

func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
		"style='stroke: grey; stroke-width: 0.7' "+
		"width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay, az, ok1 := corner(i+1, j)
			bx, by, bz, ok2 := corner(i, j)
			cx, cy, cz, ok3 := corner(i, j+1)
			dx, dy, dz, ok4 := corner(i+1, j+1)

			if !ok1 || !ok2 || !ok3 || !ok4 {
				continue
			}

			// Calculate average height of the polygon
			avgZ := (az + bz + cz + dz) / 4
			color := getColor(avgZ)

			fmt.Printf("<polygon points='%g,%g %g,%g %g,%g %g,%g' fill='%s'/>\n",
				ax, ay, bx, by, cx, cy, dx, dy, color)
		}
	}
	fmt.Println("</svg>")
}

func corner(i, j int) (float64, float64, float64, bool) {
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)

	z := f(x, y)

	if !math.IsFinite(z) {
		return 0, 0, 0, false
	}

	sx := width/2 + (x-y)*cos30*xyscale
	sy := height/2 + (x+y)*sin30*xyscale - z*zscale

	if !math.IsFinite(sx) || !math.IsFinite(sy) {
		return 0, 0, 0, false
	}

	return sx, sy, z, true
}

func getColor(z float64) string {
	min, max := -1.0, 1.0
	normalized := (z - min) / (max - min)

	if normalized < 0 {
		normalized = 0
	}
	if normalized > 1 {
		normalized = 1
	}

	red := int(normalized * 255)
	blue := int((1 - normalized) * 255)

	return fmt.Sprintf("#%02x00%02x", red, blue)
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y)
	return math.Sin(r) / r
}
```

### Exercise 3.4: Following the approach of the Lissajous example in Section 1.7, construct a web server that computes surfaces and writes SVG data to the client. The server must set the Content-Type header like this: `w.Header().Set("Content-Type", "image/svg+xml")`

```go
package main

import (
	"fmt"
	"log"
	"math"
	"net/http"
)

const (
	width, height = 600, 320
	cells         = 100
	xyrange       = 30.0
	xyscale       = width / 2 / xyrange
	zscale        = height * 0.4
	angle         = math.Pi / 6
)

var sin30, cos30 = math.Sin(angle), math.Cos(angle)

func main() {
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func handler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "image/svg+xml")
	fmt.Fprintf(w, "<svg xmlns='http://www.w3.org/2000/svg' "+
		"style='stroke: grey; fill: white; stroke-width: 0.7' "+
		"width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay, ok1 := corner(i+1, j)
			bx, by, ok2 := corner(i, j)
			cx, cy, ok3 := corner(i, j+1)
			dx, dy, ok4 := corner(i+1, j+1)

			if !ok1 || !ok2 || !ok3 || !ok4 {
				continue
			}

			fmt.Fprintf(w, "<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",
				ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Fprintln(w, "</svg>")
}

func corner(i, j int) (float64, float64, bool) {
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)

	z := f(x, y)

	if !math.IsFinite(z) {
		return 0, 0, false
	}

	sx := width/2 + (x-y)*cos30*xyscale
	sy := height/2 + (x+y)*sin30*xyscale - z*zscale

	if !math.IsFinite(sx) || !math.IsFinite(sy) {
		return 0, 0, false
	}

	return sx, sy, true
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y)
	return math.Sin(r) / r
}
```

### Exercise 3.5: Implement a full-color Mandelbrot set using the function image.NewRGBA and the type color.RGBA or color.YCbCr.

```go
package main

import (
	"image"
	"image/color"
	"image/png"
	"math/cmplx"
	"os"
)

func main() {
	const (
		xmin, ymin, xmax, ymax = -2, -2, +2, +2
		width, height          = 1024, 1024
	)

	img := image.NewRGBA(image.Rect(0, 0, width, height))
	for py := 0; py < height; py++ {
		y := float64(py)/height*(ymax-ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px)/width*(xmax-xmin) + xmin
			z := complex(x, y)
			img.Set(px, py, mandelbrot(z))
		}
	}
	png.Encode(os.Stdout, img)
}

func mandelbrot(z complex128) color.Color {
	const iterations = 200
	const contrast = 15

	var v complex128
	for n := uint8(0); n < iterations; n++ {
		v = v*v + z
		if cmplx.Abs(v) > 2 {
			// Create colorful gradient based on iteration count
			return color.RGBA{
				R: 255 - contrast*n,
				G: contrast * n,
				B: 128 + contrast*n/2,
				A: 255,
			}
		}
	}
	return color.Black
}
```

### Exercise 3.6: Supersampling is a technique to reduce the effect of pixelation by computing the color value at several points wit hin each pixel and taking the average. The simplest method is to divide each pixel into four ‘‘subpixels. ’’ Implement it.

```go
package main

import (
	"image"
	"image/color"
	"image/png"
	"math/cmplx"
	"os"
)

func main() {
	const (
		xmin, ymin, xmax, ymax = -2, -2, +2, +2
		width, height          = 1024, 1024
	)

	img := image.NewRGBA(image.Rect(0, 0, width, height))
	for py := 0; py < height; py++ {
		y := float64(py)/height*(ymax-ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px)/width*(xmax-xmin) + xmin
			img.Set(px, py, supersample(x, y, width, height, xmin, xmax, ymin, ymax))
		}
	}
	png.Encode(os.Stdout, img)
}

func supersample(x, y float64, width, height int, xmin, xmax, ymin, ymax float64) color.Color {
	offsets := []float64{-0.25, 0.25}
	var r, g, b, a uint32

	dx := (xmax - xmin) / float64(width)
	dy := (ymax - ymin) / float64(height)

	for _, ox := range offsets {
		for _, oy := range offsets {
			z := complex(x+ox*dx, y+oy*dy)
			c := mandelbrot(z)
			r1, g1, b1, a1 := c.RGBA()
			r += r1
			g += g1
			b += b1
			a += a1
		}
	}

	return color.RGBA{
		R: uint8(r / 4 / 257),
		G: uint8(g / 4 / 257),
		B: uint8(b / 4 / 257),
		A: uint8(a / 4 / 257),
	}
}

func mandelbrot(z complex128) color.Color {
	const iterations = 200
	const contrast = 15

	var v complex128
	for n := uint8(0); n < iterations; n++ {
		v = v*v + z
		if cmplx.Abs(v) > 2 {
			return color.RGBA{
				R: 255 - contrast*n,
				G: contrast * n,
				B: 128 + contrast*n/2,
				A: 255,
			}
		}
	}
	return color.Black
}
```

### Exercise 3.7: Another simple fractal uses Newton’s method to find complex solutions to a function such as z4−1 = 0. Shade each starting point by the number of iterations required to et close to one of the four roots. Color each point by the root it approaches.

```go
package main

import (
	"image"
	"image/color"
	"image/png"
	"math/cmplx"
	"os"
)

func main() {
	const (
		xmin, ymin, xmax, ymax = -2, -2, +2, +2
		width, height          = 1024, 1024
	)

	img := image.NewRGBA(image.Rect(0, 0, width, height))
	for py := 0; py < height; py++ {
		y := float64(py)/height*(ymax-ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px)/width*(xmax-xmin) + xmin
			z := complex(x, y)
			img.Set(px, py, newton(z))
		}
	}
	png.Encode(os.Stdout, img)
}

func f(z complex128) complex128 {
	return z*z*z*z - 1
}

func fPrime(z complex128) complex128 {
	return 4 * z * z * z
}

func newton(z complex128) color.Color {
	const iterations = 50
	const tolerance = 1e-6

	roots := []complex128{
		complex(1, 0),
		complex(-1, 0),
		complex(0, 1),
		complex(0, -1),
	}

	colors := []color.RGBA{
		{255, 0, 0, 255},   // Red
		{0, 255, 0, 255},   // Green
		{0, 0, 255, 255},   // Blue
		{255, 255, 0, 255}, // Yellow
	}

	for n := 0; n < iterations; n++ {
		z = z - f(z)/fPrime(z)

		for i, root := range roots {
			if cmplx.Abs(z-root) < tolerance {
				shade := uint8(255 - (255 * n / iterations))
				c := colors[i]
				return color.RGBA{
					R: uint8(int(c.R) * int(shade) / 255),
					G: uint8(int(c.G) * int(shade) / 255),
					B: uint8(int(c.B) * int(shade) / 255),
					A: 255,
				}
			}
		}
	}

	return color.Black
}
```

### Exercise 3.8: Render ing fractals at high zoom levels demands great arithmetic precision. Implement the same fractal using four different representations of numbers: complex64, complex128, big.Float, and big.Rat. (The latter two types are found in the math/big package. Float uses arbitrary but bounded-precision floating-point; Rat uses unbounded-precision rational numbers.) How do they comp are in performance and memory usage? At what zoom levels do render ing artifacts become visible?

```go
package main

import (
	"image"
	"image/color"
	"image/png"
	"math/big"
	"math/cmplx"
	"os"
)

func mandelbrot64(z complex64) color.Color {
	const iterations = 200
	const contrast = 15

	var v complex64
	for n := uint8(0); n < iterations; n++ {
		v = v*v + z
		if real(v)*real(v)+imag(v)*imag(v) > 4 {
			return color.Gray{255 - contrast*n}
		}
	}
	return color.Black
}

func mandelbrot128(z complex128) color.Color {
	const iterations = 200
	const contrast = 15

	var v complex128
	for n := uint8(0); n < iterations; n++ {
		v = v*v + z
		if cmplx.Abs(v) > 2 {
			return color.Gray{255 - contrast*n}
		}
	}
	return color.Black
}

func mandelbrotBigFloat(zReal, zImag *big.Float) color.Color {
	const iterations = 200
	const contrast = 15

	vReal := new(big.Float)
	vImag := new(big.Float)

	two := big.NewFloat(2)
	four := big.NewFloat(4)

	for n := uint8(0); n < iterations; n++ {
		vReal2 := new(big.Float).Mul(vReal, vReal)
		vImag2 := new(big.Float).Mul(vImag, vImag)

		newVReal := new(big.Float).Sub(vReal2, vImag2)
		newVReal.Add(newVReal, zReal)

		newVImag := new(big.Float).Mul(vReal, vImag)
		newVImag.Mul(newVImag, two)
		newVImag.Add(newVImag, zImag)

		vReal = newVReal
		vImag = newVImag

		absSquared := new(big.Float).Add(vReal2, vImag2)
		if absSquared.Cmp(four) > 0 {
			return color.Gray{255 - contrast*n}
		}
	}
	return color.Black
}

func mandelbrotBigRat(zReal, zImag *big.Rat) color.Color {
	const iterations = 200
	const contrast = 15

	vReal := new(big.Rat)
	vImag := new(big.Rat)

	four := big.NewRat(4, 1)
	two := big.NewRat(2, 1)

	for n := uint8(0); n < iterations; n++ {
		vReal2 := new(big.Rat).Mul(vReal, vReal)
		vImag2 := new(big.Rat).Mul(vImag, vImag)

		newVReal := new(big.Rat).Sub(vReal2, vImag2)
		newVReal.Add(newVReal, zReal)

		newVImag := new(big.Rat).Mul(vReal, vImag)
		newVImag.Mul(newVImag, two)
		newVImag.Add(newVImag, zImag)

		vReal = newVReal
		vImag = newVImag

		absSquared := new(big.Rat).Add(vReal2, vImag2)
		if absSquared.Cmp(four) > 0 {
			return color.Gray{255 - contrast*n}
		}
	}
	return color.Black
}

func main() {
	const (
		xmin, ymin, xmax, ymax = -2, -2, +2, +2
		width, height          = 1024, 1024
	)

	img := image.NewRGBA(image.Rect(0, 0, width, height))
	for py := 0; py < height; py++ {
		y := float64(py)/height*(ymax-ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px)/width*(xmax-xmin) + xmin
			z := complex(x, y)
			img.Set(px, py, mandelbrot128(z))
		}
	}
	png.Encode(os.Stdout, img)
}
```

### Exercise 3.9: Write a web server that renders fractals and writes the image data to the client. Allow the client to specify the x, y, and zoom values as parameters to the HTTP request.

```go
package main

import (
	"image"
	"image/color"
	"image/png"
	"log"
	"math/cmplx"
	"net/http"
	"strconv"
)

func main() {
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func handler(w http.ResponseWriter, r *http.Request) {
	x, _ := strconv.ParseFloat(r.URL.Query().Get("x"), 64)
	y, _ := strconv.ParseFloat(r.URL.Query().Get("y"), 64)
	zoom, _ := strconv.ParseFloat(r.URL.Query().Get("zoom"), 64)

	if zoom == 0 {
		zoom = 1
	}

	const (
		width, height = 1024, 1024
	)

	scale := 4.0 / zoom
	xmin := x - scale/2
	xmax := x + scale/2
	ymin := y - scale/2
	ymax := y + scale/2

	img := image.NewRGBA(image.Rect(0, 0, width, height))
	for py := 0; py < height; py++ {
		yCoord := float64(py)/height*(ymax-ymin) + ymin
		for px := 0; px < width; px++ {
			xCoord := float64(px)/width*(xmax-xmin) + xmin
			z := complex(xCoord, yCoord)
			img.Set(px, py, mandelbrot(z))
		}
	}

	w.Header().Set("Content-Type", "image/png")
	png.Encode(w, img)
}

func mandelbrot(z complex128) color.Color {
	const iterations = 200
	const contrast = 15

	var v complex128
	for n := uint8(0); n < iterations; n++ {
		v = v*v + z
		if cmplx.Abs(v) > 2 {
			return color.Gray{255 - contrast*n}
		}
	}
	return color.Black
}
```

### Exercise 3.10: Write a non-rec ursive version of comma, using bytes.Buffer instead of string concatenation.

```go
package main

import (
	"bytes"
	"fmt"
)

func comma(s string) string {
	var buf bytes.Buffer
	n := len(s)

	prefix := n % 3
	if prefix == 0 {
		prefix = 3
	}

	buf.WriteString(s[:prefix])

	for i := prefix; i < n; i += 3 {
		buf.WriteByte(',')
		buf.WriteString(s[i : i+3])
	}

	return buf.String()
}
```

### Exercise 3.11: Enhance comma so that it deals correctly with floating-point numbers and an optional sign.

```go
package main

import (
	"bytes"
	"fmt"
	"strings"
)

func comma(s string) string {
	var buf bytes.Buffer

	start := 0
	if s[0] == '+' || s[0] == '-' {
		buf.WriteByte(s[0])
		start = 1
	}

	parts := strings.Split(s[start:], ".")
	intPart := parts[0]

	n := len(intPart)
	prefix := n % 3
	if prefix == 0 && n > 0 {
		prefix = 3
	}

	if prefix > 0 {
		buf.WriteString(intPart[:prefix])
	}

	for i := prefix; i < n; i += 3 {
		if i > 0 {
			buf.WriteByte(',')
		}
		buf.WriteString(intPart[i : i+3])
	}

	if len(parts) > 1 {
		buf.WriteByte('.')
		buf.WriteString(parts[1])
	}

	return buf.String()
}
```

### Exercise 3.12: Write a function that rep orts whether two strings are anagrams of each other, that is, they contain the same letters in a different order.

```go
package main

import (
	"fmt"
)

func areAnagrams(s1, s2 string) bool {
	if len(s1) != len(s2) {
		return false
	}

	counts := make(map[rune]int)

	for _, r := range s1 {
		counts[r]++
	}

	for _, r := range s2 {
		counts[r]--
		if counts[r] < 0 {
			return false
		}
	}

	for _, count := range counts {
		if count != 0 {
			return false
		}
	}

	return true
}
```

### Exercise 3.13: Write const declarations for KB, MB, up through YB as compactly as you can.

```go
package main

const (
	KB = 1000
	MB = KB * 1000
	GB = MB * 1000
	TB = GB * 1000
	PB = TB * 1000
	EB = PB * 1000
	ZB = EB * 1000
	YB = ZB * 1000
)

const (
	_ = 1 << (10 * iota)
	KiB
	MiB
	GiB
	TiB
	PiB
	EiB
	ZiB
	YiB
)
```
