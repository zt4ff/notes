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

## Chapter 4

### Exercise 4.1: Write a function that counts the number of bits that are different in two SHA256 hashes. (See PopCount from Section 2.6.2.)

```go
package main

import (
	"crypto/sha256"
	"fmt"
)

func hammingDistance(hash1, hash2 [32]byte) int {
	count := 0
	for i := 0; i < 32; i++ {
		xor := hash1[i] ^ hash2[i]
		for xor != 0 {
			xor &= xor - 1
			count++
		}
	}
	return count
}

func main() {
	s1 := "hello"
	s2 := "Hello"

	hash1 := sha256.Sum256([]byte(s1))
	hash2 := sha256.Sum256([]byte(s2))

	fmt.Printf("Hamming distance: %d bits\n", hammingDistance(hash1, hash2))
}
```

### Exercise 4.2: Write a program that prints the SHA256 hash of its standard input by default but supports a command-line flag to print the SHA384 or SHA512 hash instead.

```go
package main

import (
	"crypto/sha256"
	"crypto/sha512"
	"flag"
	"fmt"
	"io"
	"os"
)

func main() {
	hashType := flag.String("hash", "sha256", "hash type: sha256, sha384, or sha512")
	flag.Parse()

	data, err := io.ReadAll(os.Stdin)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error reading input: %v\n", err)
		os.Exit(1)
	}

	switch *hashType {
	case "sha256":
		fmt.Printf("%x\n", sha256.Sum256(data))
	case "sha384":
		fmt.Printf("%x\n", sha512.Sum384(data))
	case "sha512":
		fmt.Printf("%x\n", sha512.Sum512(data))
	default:
		fmt.Fprintf(os.Stderr, "unsupported hash type: %s\n", *hashType)
		os.Exit(1)
	}
}
```

### Exercise 4.3: Rewrite reverse to use an array pointer instead of a slice.

```go
package main

import "fmt"

func reverse(arr *[5]int) {
	for i, j := 0, len(arr)-1; i < j; i, j = i+1, j-1 {
		arr[i], arr[j] = arr[j], arr[i]
	}
}

func main() {
	a := [5]int{1, 2, 3, 4, 5}
	fmt.Println(a)
	reverse(&a)
	fmt.Println(a)
}
```

### Exercise 4.4: Write a version of rotate that operates in a single pass.

```go
package main

import "fmt"

func rotate(s []int, n int) {
	n = n % len(s)
	if n < 0 {
		n += len(s)
	}

	reverse(s[:n])
	reverse(s[n:])
	reverse(s)
}

func reverse(s []int) {
	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
}

func main() {
	s := []int{1, 2, 3, 4, 5}
	rotate(s, 2)
	fmt.Println(s)
}
```

### Exercise 4.5: Write an in-place function to eliminate adjacent duplicates in a []string slice.

```go
package main

import "fmt"

func removeDuplicates(s []string) []string {
	if len(s) == 0 {
		return s
	}

	j := 0
	for i := 1; i < len(s); i++ {
		if s[i] != s[j] {
			j++
			s[j] = s[i]
		}
	}
	return s[:j+1]
}

func main() {
	s := []string{"a", "a", "b", "b", "b", "c", "a", "a"}
	fmt.Println(removeDuplicates(s))
}
```

### Exercise 4.6: Write an in-place function that squashes each run of adjacent Unicode spaces (see unicode.IsSpace) in a UTF-8-enco ded []byte slice into a single ASCII space.

```go
package main

import (
	"fmt"
	"unicode"
	"unicode/utf8"
)

func squashSpaces(s []byte) []byte {
	out := s[:0]
	inSpace := false

	for i := 0; i < len(s); {
		r, size := utf8.DecodeRune(s[i:])
		if unicode.IsSpace(r) {
			if !inSpace {
				out = append(out, ' ')
				inSpace = true
			}
			i += size
		} else {
			out = append(out, s[i:i+size]...)
			inSpace = false
			i += size
		}
	}

	return out
}

func main() {
	s := []byte("hello   world\t\nfoo  bar")
	fmt.Printf("%q\n", s)
	s = squashSpaces(s)
	fmt.Printf("%q\n", s)
}
```

### Exercise 4.7: Modify reverse to reverse the characters of a []byte slice that represents a UTF-8-encoded string, in place. Can you do it without allocat ing new memory?

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func reverseUTF8(s []byte) {
	for i := 0; i < len(s); {
		_, size := utf8.DecodeRune(s[i:])
		reverse(s[i : i+size])
		i += size
	}
	reverse(s)
}

func reverse(s []byte) {
	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
}

func main() {
	s := []byte("Hello, 世界")
	fmt.Printf("%s\n", s)
	reverseUTF8(s)
	fmt.Printf("%s\n", s)
}
```

### Exercise 4.8: Modify charcount to count letters, digits, and so on in their Unicode categories, using functions like unicode.IsLetter.

```go
.package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"unicode"
)

func main() {
	counts := make(map[rune]int)
	letters := 0
	digits := 0
	spaces := 0
	marks := 0
	punctuation := 0
	symbols := 0
	other := 0
	invalid := 0

	in := bufio.NewReader(os.Stdin)
	for {
		r, n, err := in.ReadRune()
		if err == io.EOF {
			break
		}
		if err != nil {
			fmt.Fprintf(os.Stderr, "charcount: %v\n", err)
			os.Exit(1)
		}
		if r == unicode.ReplacementChar && n == 1 {
			invalid++
			continue
		}
		counts[r]++

		switch {
		case unicode.IsLetter(r):
			letters++
		case unicode.IsDigit(r):
			digits++
		case unicode.IsSpace(r):
			spaces++
		case unicode.IsMark(r):
			marks++
		case unicode.IsPunct(r):
			punctuation++
		case unicode.IsSymbol(r):
			symbols++
		default:
			other++
		}
	}

	fmt.Println("Categories:")
	fmt.Printf("letters:\t%d\n", letters)
	fmt.Printf("digits:\t\t%d\n", digits)
	fmt.Printf("spaces:\t\t%d\n", spaces)
	fmt.Printf("marks:\t\t%d\n", marks)
	fmt.Printf("punctuation:\t%d\n", punctuation)
	fmt.Printf("symbols:\t%d\n", symbols)
	fmt.Printf("other:\t\t%d\n", other)
	if invalid > 0 {
		fmt.Printf("invalid:\t%d\n", invalid)
	}
}
```

### Exercise 4.9: Write a program wordfreq to rep ort the frequency of each word in an input text file. Call input.Split(bufio.ScanWords) before the first call to Scan to break the input into words instead of lines.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "usage: wordfreq <file>\n")
		os.Exit(1)
	}

	file, err := os.Open(os.Args[1])
	if err != nil {
		fmt.Fprintf(os.Stderr, "wordfreq: %v\n", err)
		os.Exit(1)
	}
	defer file.Close()

	counts := make(map[string]int)
	input := bufio.NewScanner(file)
	input.Split(bufio.ScanWords)

	for input.Scan() {
		counts[input.Text()]++
	}

	if err := input.Err(); err != nil {
		fmt.Fprintf(os.Stderr, "wordfreq: %v\n", err)
		os.Exit(1)
	}

	for word, count := range counts {
		fmt.Printf("%s\t%d\n", word, count)
	}
}
```

### Exercise 4.10: Modify issues to report the results in age categories, say less than a month old, less than a year old, and more than a year old.

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"net/url"
	"os"
	"time"
)

const IssuesURL = "https://api.github.com/search/issues"

type IssuesSearchResult struct {
	TotalCount int `json:"total_count"`
	Items      []*Issue
}

type Issue struct {
	Number    int
	HTMLURL   string `json:"html_url"`
	Title     string
	State     string
	User      *User
	CreatedAt time.Time `json:"created_at"`
}

type User struct {
	Login   string
	HTMLURL string `json:"html_url"`
}

func SearchIssues(terms []string) (*IssuesSearchResult, error) {
	q := url.QueryEscape(fmt.Sprintf("%v", terms))
	resp, err := http.Get(IssuesURL + "?q=" + q)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("search query failed: %s", resp.Status)
	}

	var result IssuesSearchResult
	if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
		return nil, err
	}
	return &result, nil
}

func main() {
	result, err := SearchIssues(os.Args[1:])
	if err != nil {
		log.Fatal(err)
	}

	now := time.Now()
	var lessThanMonth, lessThanYear, moreThanYear []*Issue

	for _, item := range result.Items {
		age := now.Sub(item.CreatedAt)
		if age < 30*24*time.Hour {
			lessThanMonth = append(lessThanMonth, item)
		} else if age < 365*24*time.Hour {
			lessThanYear = append(lessThanYear, item)
		} else {
			moreThanYear = append(moreThanYear, item)
		}
	}

	fmt.Printf("%d issues:\n", result.TotalCount)

	fmt.Println("\nLess than a month old:")
	for _, item := range lessThanMonth {
		fmt.Printf("#%-5d %9.9s %.55s\n", item.Number, item.User.Login, item.Title)
	}

	fmt.Println("\nLess than a year old:")
	for _, item := range lessThanYear {
		fmt.Printf("#%-5d %9.9s %.55s\n", item.Number, item.User.Login, item.Title)
	}

	fmt.Println("\nMore than a year old:")
	for _, item := range moreThanYear {
		fmt.Printf("#%-5d %9.9s %.55s\n", item.Number, item.User.Login, item.Title)
	}
}
```

### Exercise 4.11: Build a tool that lets users create, read, update, and close GitHub issues from the command line, invoking their preferred text editor when subst antial text input is required.

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"os/exec"
	"strings"
)

const APIBase = "https://api.github.com"

type Issue struct {
	Number int    `json:"number,omitempty"`
	Title  string `json:"title"`
	Body   string `json:"body"`
	State  string `json:"state,omitempty"`
}

func getToken() string {
	token := os.Getenv("GITHUB_TOKEN")
	if token == "" {
		fmt.Fprintln(os.Stderr, "GITHUB_TOKEN environment variable not set")
		os.Exit(1)
	}
	return token
}

func editInEditor() (string, error) {
	editor := os.Getenv("EDITOR")
	if editor == "" {
		editor = "vi"
	}

	tmpfile, err := os.CreateTemp("", "issue-*.txt")
	if err != nil {
		return "", err
	}
	defer os.Remove(tmpfile.Name())
	tmpfile.Close()

	cmd := exec.Command(editor, tmpfile.Name())
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		return "", err
	}

	content, err := os.ReadFile(tmpfile.Name())
	if err != nil {
		return "", err
	}

	return string(content), nil
}

func createIssue(owner, repo, title, body string) error {
	issue := Issue{Title: title, Body: body}
	data, err := json.Marshal(issue)
	if err != nil {
		return err
	}

	url := fmt.Sprintf("%s/repos/%s/%s/issues", APIBase, owner, repo)
	req, err := http.NewRequest("POST", url, bytes.NewBuffer(data))
	if err != nil {
		return err
	}

	req.Header.Set("Authorization", "token "+getToken())
	req.Header.Set("Content-Type", "application/json")

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusCreated {
		body, _ := io.ReadAll(resp.Body)
		return fmt.Errorf("failed to create issue: %s\n%s", resp.Status, body)
	}

	fmt.Println("Issue created successfully")
	return nil
}

func readIssue(owner, repo, number string) error {
	url := fmt.Sprintf("%s/repos/%s/%s/issues/%s", APIBase, owner, repo, number)
	req, err := http.NewRequest("GET", url, nil)
	if err != nil {
		return err
	}

	req.Header.Set("Authorization", "token "+getToken())

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return fmt.Errorf("failed to read issue: %s", resp.Status)
	}

	var issue Issue
	if err := json.NewDecoder(resp.Body).Decode(&issue); err != nil {
		return err
	}

	fmt.Printf("Issue #%d\nTitle: %s\nState: %s\n\n%s\n", issue.Number, issue.Title, issue.State, issue.Body)
	return nil
}

func updateIssue(owner, repo, number, title, body string) error {
	issue := Issue{Title: title, Body: body}
	data, err := json.Marshal(issue)
	if err != nil {
		return err
	}

	url := fmt.Sprintf("%s/repos/%s/%s/issues/%s", APIBase, owner, repo, number)
	req, err := http.NewRequest("PATCH", url, bytes.NewBuffer(data))
	if err != nil {
		return err
	}

	req.Header.Set("Authorization", "token "+getToken())
	req.Header.Set("Content-Type", "application/json")

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		body, _ := io.ReadAll(resp.Body)
		return fmt.Errorf("failed to update issue: %s\n%s", resp.Status, body)
	}

	fmt.Println("Issue updated successfully")
	return nil
}

func closeIssue(owner, repo, number string) error {
	issue := Issue{State: "closed"}
	data, err := json.Marshal(issue)
	if err != nil {
		return err
	}

	url := fmt.Sprintf("%s/repos/%s/%s/issues/%s", APIBase, owner, repo, number)
	req, err := http.NewRequest("PATCH", url, bytes.NewBuffer(data))
	if err != nil {
		return err
	}

	req.Header.Set("Authorization", "token "+getToken())
	req.Header.Set("Content-Type", "application/json")

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return fmt.Errorf("failed to close issue: %s", resp.Status)
	}

	fmt.Println("Issue closed successfully")
	return nil
}

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "usage: issues <command> [args]\n")
		fmt.Fprintf(os.Stderr, "commands:\n")
		fmt.Fprintf(os.Stderr, "  create <owner/repo> <title>\n")
		fmt.Fprintf(os.Stderr, "  read <owner/repo> <number>\n")
		fmt.Fprintf(os.Stderr, "  update <owner/repo> <number> <title>\n")
		fmt.Fprintf(os.Stderr, "  close <owner/repo> <number>\n")
		os.Exit(1)
	}

	command := os.Args[1]

	switch command {
	case "create":
		if len(os.Args) < 4 {
			fmt.Fprintln(os.Stderr, "usage: issues create <owner/repo> <title>")
			os.Exit(1)
		}
		parts := strings.Split(os.Args[2], "/")
		if len(parts) != 2 {
			fmt.Fprintln(os.Stderr, "repo must be in format owner/repo")
			os.Exit(1)
		}
		title := os.Args[3]
		body, err := editInEditor()
		if err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}
		if err := createIssue(parts[0], parts[1], title, body); err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}

	case "read":
		if len(os.Args) < 4 {
			fmt.Fprintln(os.Stderr, "usage: issues read <owner/repo> <number>")
			os.Exit(1)
		}
		parts := strings.Split(os.Args[2], "/")
		if len(parts) != 2 {
			fmt.Fprintln(os.Stderr, "repo must be in format owner/repo")
			os.Exit(1)
		}
		if err := readIssue(parts[0], parts[1], os.Args[3]); err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}

	case "update":
		if len(os.Args) < 5 {
			fmt.Fprintln(os.Stderr, "usage: issues update <owner/repo> <number> <title>")
			os.Exit(1)
		}
		parts := strings.Split(os.Args[2], "/")
		if len(parts) != 2 {
			fmt.Fprintln(os.Stderr, "repo must be in format owner/repo")
			os.Exit(1)
		}
		title := os.Args[4]
		body, err := editInEditor()
		if err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}
		if err := updateIssue(parts[0], parts[1], os.Args[3], title, body); err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}

	case "close":
		if len(os.Args) < 4 {
			fmt.Fprintln(os.Stderr, "usage: issues close <owner/repo> <number>")
			os.Exit(1)
		}
		parts := strings.Split(os.Args[2], "/")
		if len(parts) != 2 {
			fmt.Fprintln(os.Stderr, "repo must be in format owner/repo")
			os.Exit(1)
		}
		if err := closeIssue(parts[0], parts[1], os.Args[3]); err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}

	default:
		fmt.Fprintf(os.Stderr, "unknown command: %s\n", command)
		os.Exit(1)
	}
}
```

### Exercise 4.12: The popu lar web comic xkcd has a JSON interface. For example, a request to https://xkcd.com/571/info.0.json produces a detailed description of comic 571, one of many favorites. Download each URL (once!) and build an offline index. Write a tool xkcd that, using this index, prints the URL and transcript of each comic that matches a search term provided on the command line.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"path/filepath"
	"strconv"
	"strings"
)

const (
	indexDir  = "xkcd_index"
	maxComics = 3000
)

type Comic struct {
	Num        int    `json:"num"`
	Title      string `json:"title"`
	Transcript string `json:"transcript"`
	Alt        string `json:"alt"`
	Img        string `json:"img"`
}

func downloadComic(num int) (*Comic, error) {
	url := fmt.Sprintf("https://xkcd.com/%d/info.0.json", num)
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("status: %s", resp.Status)
	}

	var comic Comic
	if err := json.NewDecoder(resp.Body).Decode(&comic); err != nil {
		return nil, err
	}

	return &comic, nil
}

func saveComic(comic *Comic) error {
	if err := os.MkdirAll(indexDir, 0755); err != nil {
		return err
	}

	filename := filepath.Join(indexDir, fmt.Sprintf("%d.json", comic.Num))
	data, err := json.MarshalIndent(comic, "", "  ")
	if err != nil {
		return err
	}

	return os.WriteFile(filename, data, 0644)
}

func loadComic(num int) (*Comic, error) {
	filename := filepath.Join(indexDir, fmt.Sprintf("%d.json", num))
	data, err := os.ReadFile(filename)
	if err != nil {
		return nil, err
	}

	var comic Comic
	if err := json.Unmarshal(data, &comic); err != nil {
		return nil, err
	}

	return &comic, nil
}

func buildIndex() error {
	fmt.Println("Building index...")
	for i := 1; i <= maxComics; i++ {
		filename := filepath.Join(indexDir, fmt.Sprintf("%d.json", i))
		if _, err := os.Stat(filename); err == nil {
			continue
		}

		comic, err := downloadComic(i)
		if err != nil {
			continue
		}

		if err := saveComic(comic); err != nil {
			return err
		}

		fmt.Printf("Downloaded comic %d\n", i)
	}
	fmt.Println("Index built successfully")
	return nil
}

func search(term string) error {
	files, err := os.ReadDir(indexDir)
	if err != nil {
		return fmt.Errorf("index not found, run with 'build' command first")
	}

	term = strings.ToLower(term)
	found := false

	for _, file := range files {
		if filepath.Ext(file.Name()) != ".json" {
			continue
		}

		numStr := strings.TrimSuffix(file.Name(), ".json")
		num, err := strconv.Atoi(numStr)
		if err != nil {
			continue
		}

		comic, err := loadComic(num)
		if err != nil {
			continue
		}

		if strings.Contains(strings.ToLower(comic.Title), term) ||
			strings.Contains(strings.ToLower(comic.Transcript), term) ||
			strings.Contains(strings.ToLower(comic.Alt), term) {
			fmt.Printf("Comic #%d: %s\n", comic.Num, comic.Title)
			fmt.Printf("URL: https://xkcd.com/%d/\n", comic.Num)
			fmt.Printf("Transcript: %s\n\n", comic.Transcript)
			found = true
		}
	}

	if !found {
		fmt.Println("No comics found matching:", term)
	}

	return nil
}

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "usage: xkcd <command> [args]\n")
		fmt.Fprintf(os.Stderr, "commands:\n")
		fmt.Fprintf(os.Stderr, "  build          - build the index\n")
		fmt.Fprintf(os.Stderr, "  search <term>  - search for comics\n")
		os.Exit(1)
	}

	command := os.Args[1]

	switch command {
	case "build":
		if err := buildIndex(); err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}
	case "search":
		if len(os.Args) < 3 {
			fmt.Fprintf(os.Stderr, "usage: xkcd search <term>\n")
			os.Exit(1)
		}
		if err := search(os.Args[2]); err != nil {
			fmt.Fprintf(os.Stderr, "error: %v\n", err)
			os.Exit(1)
		}
	default:
		fmt.Fprintf(os.Stderr, "unknown command: %s\n", command)
		os.Exit(1)
	}
}
```

### Exercise 4.13: The JSON-based web service of the Open Movie Databas e lets you search https://omdbapi.com/ for a movie by name and download its poster image. Write a tool poster that downloads the poster image for the movie named on the command line.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"os"
	"strings"
)

const OMDBAPI = "https://www.omdbapi.com/"

type Movie struct {
	Title  string `json:"Title"`
	Poster string `json:"Poster"`
}

func searchMovie(title, apiKey string) (*Movie, error) {
	params := url.Values{}
	params.Add("t", title)
	params.Add("apikey", apiKey)

	resp, err := http.Get(OMDBAPI + "?" + params.Encode())
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("search failed: %s", resp.Status)
	}

	var movie Movie
	if err := json.NewDecoder(resp.Body).Decode(&movie); err != nil {
		return nil, err
	}

	if movie.Poster == "" || movie.Poster == "N/A" {
		return nil, fmt.Errorf("no poster available for: %s", title)
	}

	return &movie, nil
}

func downloadPoster(posterURL, filename string) error {
	resp, err := http.Get(posterURL)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return fmt.Errorf("download failed: %s", resp.Status)
	}

	file, err := os.Create(filename)
	if err != nil {
		return err
	}
	defer file.Close()

	_, err = io.Copy(file, resp.Body)
	return err
}

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "usage: poster <movie title>\n")
		os.Exit(1)
	}

	apiKey := os.Getenv("OMDB_API_KEY")
	if apiKey == "" {
		fmt.Fprintf(os.Stderr, "OMDB_API_KEY environment variable not set\n")
		fmt.Fprintf(os.Stderr, "Get a free API key from https://www.omdbapi.com/apikey.aspx\n")
		os.Exit(1)
	}

	title := strings.Join(os.Args[1:], " ")

	movie, err := searchMovie(title, apiKey)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}

	filename := strings.ReplaceAll(movie.Title, " ", "_") + ".jpg"

	if err := downloadPoster(movie.Poster, filename); err != nil {
		fmt.Fprintf(os.Stderr, "error downloading poster: %v\n", err)
		os.Exit(1)
	}

	fmt.Printf("Poster for '%s' saved as %s\n", movie.Title, filename)
}
```

### Exercise 4.14: Create a web server that quer ies GitHub once and then allows navigation of the list of bug rep orts, milestones, and users.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"os"
	"strings"
)

const OMDBAPI = "https://www.omdbapi.com/"

type Movie struct {
	Title  string `json:"Title"`
	Poster string `json:"Poster"`
}

func searchMovie(title, apiKey string) (*Movie, error) {
	params := url.Values{}
	params.Add("t", title)
	params.Add("apikey", apiKey)

	resp, err := http.Get(OMDBAPI + "?" + params.Encode())
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("search failed: %s", resp.Status)
	}

	var movie Movie
	if err := json.NewDecoder(resp.Body).Decode(&movie); err != nil {
		return nil, err
	}

	if movie.Poster == "" || movie.Poster == "N/A" {
		return nil, fmt.Errorf("no poster available for: %s", title)
	}

	return &movie, nil
}

func downloadPoster(posterURL, filename string) error {
	resp, err := http.Get(posterURL)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return fmt.Errorf("download failed: %s", resp.Status)
	}

	file, err := os.Create(filename)
	if err != nil {
		return err
	}
	defer file.Close()

	_, err = io.Copy(file, resp.Body)
	return err
}

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "usage: poster <movie title>\n")
		os.Exit(1)
	}

	apiKey := os.Getenv("OMDB_API_KEY")
	if apiKey == "" {
		fmt.Fprintf(os.Stderr, "OMDB_API_KEY environment variable not set\n")
		fmt.Fprintf(os.Stderr, "Get a free API key from https://www.omdbapi.com/apikey.aspx\n")
		os.Exit(1)
	}

	title := strings.Join(os.Args[1:], " ")

	movie, err := searchMovie(title, apiKey)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}

	filename := strings.ReplaceAll(movie.Title, " ", "_") + ".jpg"

	if err := downloadPoster(movie.Poster, filename); err != nil {
		fmt.Fprintf(os.Stderr, "error downloading poster: %v\n", err)
		os.Exit(1)
	}

	fmt.Printf("Poster for '%s' saved as %s\n", movie.Title, filename)
}
```
