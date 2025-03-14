---
date: 2024-11-06
slug: steal-this-class
categories:
  - C++
  - Best Practices
---

# Steal this Class

Class design in C++ is hard. Really, really, hard. I love C++ but, sadly, all the defaults are wrong.

<!-- more -->

This means you have to add a lot of “ornamentation” to a well-designed C++ class. This can seem bulky, so you might start second-guessing and questioning yourself, and it’s easy to forget a thing or two.

To make things worse, much of the advice on Stack Overflow and (LLM output trained on that advice) is outdated, incomplete, and just wrong.

Since Ctrl-C, Ctrl-V is the de facto standard programming paradigm, and you are going to do it regardless, consider using these boilerplate classes. They are “copy-paste safe.” They have the right defaults, and they are not missing some essential piece that will get you into trouble later.

<center>![Cute cat](take-this.jpg)</center>

Of course, we have the [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines). These are all the defaults and best practices you *should* be using. But, be honest, you’re not going to read those, are you? These are the templates you have been looking for.

## A Plain POD { data-toc-label="Plain POD" }

```cpp
struct Point {
  // A1 Use 'struct' for these, and only these, basic POD (Plain Old Data) types.
  //    This reduces clutter by providing public visibility by default.

  constexpr bool operator==(const Point&) const noexcept = default;
  // A2 Add the == operator (!= added implicitly) to make the type "regular"
  // A3 Make the operators free functions in C++14 & 17
  // A4 Provide explicit operator!= in C++14 & 17

  float x{};
  float y{};
  // A5 Add {} to members of primitive type to make sure they are initialized.
  // A6 Keep members after the functions. In case you need to convert it to a
  //    class, this will produce a smaller git diff (less risk for conflicts).
  // A7 Order members by "type with largest alignment" first, to keep the
  //    struct small. Doubles first, then ints, then chars.
};
```

## Basic Data Types with Invariants  { data-toc-label="Data with Invariants" }

```cpp
class Circle {
  // B1 Make anything with logic a class (not a struct)
public:
  constexpr Circle() = default;
  // B2 Always make basic data types default constructible to be "regular"
  // B3 constexpr makes the data type more useful

  constexpr explicit Circle(float radius) : Circle(Point{}, radius) {}
  // B4 Use 'explicit' for all single argument constructors to prevent
  //    unintentional implicit conversion.
  // B5 Limit duplication in constructor bodies with delegating constructors

  constexpr Circle(const Point& center, float radius) :
    center_(center), radius_(radius < 0.0f ? 0.0f : radius) {}
  // B6 Handle pre-condition violations and preserve class invariants
  //    by truncation or throwing. Throw unless constructor is constexpr.

  // B7 NO: constexpr Circle(float x, float y, float radius)
  //    Don't offer this constructor to avoid confusion with argument order
  //    at the call site, i.e. Circle{radius, x, y}

  [[nodiscard]] constexpr const Point& getCenter() const noexcept { return center_; }
  // B8  Use both constexpr and const
  // B9  Use [[nodiscard]] where it's clearly a bug to not use a return value
  // B10 All logically non-mutating functions should be const

  constexpr void setCenter(const Point& center) { center_ = center; }
  // B11 Setters can often not be noexcept since they might copy and allocate
  //     In this case it would be fine, but not in general.

  [[nodiscard]] constexpr float getRadius() const noexcept { return radius_; }

  constexpr void setRadius(float radius) { radius_ = radius < 0.0f ? 0.0f : radius; }
  // B12 Preserve invariants in setters (or throw)
  // B13 Take arguments as value in case they're small enough to be passed
  //     in a register, i.e. all built-in types and smart pointers.

  constexpr bool operator==(const Circle&) const noexcept = default;
  // B14 Add the == operator (!= added implicitly) to make the type "regular"
  // B15 Make the operators free functions in C++14 & 17
  // B16 Provide explicit operator!= in C++14 & 17

private:
  // B17 All mutable members are non const, non ref and private

  Point center_{};
  float radius_{};
  // B18 Use trailing underscore to keep good parameter names in setter
  //     function signatures.
};
```

## Resource Owning RAII Types { data-toc-label="RAII Types" }

```cpp
class File {
// C1 RAII types should be classes (not structs)

public:
  [[nodiscard]] explicit File(const std::filesystem::path& fileName)
      : file_(std::fopen(fileName.c_str(), "r")) {
    if (!file_) throw std::runtime_error("Failed to open file");
  }
  // C2 [[nodiscard]] on constructors prevents forgetting to name the RAII object.
  //    Especially important if the 'resource' is a side-effect, like a lock.
  // C3 Use exceptions to signal errors
  // C4 Make single-arg constructor 'explicit' to prevent implicit conversion.

  ~File() noexcept { reset(); }
  // C5 All destructors should be noexcept
  // C6 Not 'virtual' since it is not meant as a base class
  // C7 Use reset() function to avoid code duplication in move assignment

  File(File&& other) noexcept : file_(std::exchange(other.file_, nullptr)) {}
  // C8 Use std::exchange to leave the moved-from object in a clean state

  File& operator=(File&& other) & noexcept {
    if (this != &other) {
      reset();
      std::swap(file_, other.file_);
    }
    return *this;
  }
  // C9  Can't use copy-and-swap idiom for move only types
  // C10 reset() function to avoid code duplication here and in destructor
  // C11 Release current resource before acquiring new one
  // C12 Use std::swap leaves moved-from object in a clean state
  // C13 Make move operations noexcept to work well with std::vector, etc.
  // C14 Ref qualifier '&' prevents move assignment to rvalue
  // C15 Move assignment adds considerable complexity. Consider deleting it instead.

  File(const File&) = delete;
  File& operator=(const File&) = delete;
  // C16 RAII types should typically not be copyable. Be explicit (rule of 0/5).

private:
  void reset() noexcept {
    if (file_) {
      std::fclose(file_);
      file_ = nullptr;
    }
  }
  // C17 Avoid code duplication between dtor and move assignment operator.
  // C18 Set file_ to nullptr to avoid double closing.

  FILE* file_ = nullptr;
  // C19 Initialize to nullptr in case there's an exception before
  //     initializer-list assignment.
};
```

## Polymorphic Types

```cpp
class Polygon {
// D1 Interfaces should be classes (not structs)

public:
  virtual ~Polygon() noexcept = default;
  // D2 Add a virtual destructor to ensure derived class destructor is called
  // D3 All destructors should be noexcept

  [[nodiscard]] virtual double sumOfAngles() const noexcept = 0;
  // D4 Pure virtual interface API functions
  // D5 noexcept enforces noexcept in subclasses (use if desirable)
  // D6 [[nodiscard]] and const for getters

protected:
  Polygon() = default;
  // D7 protected constructor to prevent instantiation of interface class
  // D8 Not private to make default constructor available in subclass

private:
  Polygon(const Polygon&) = delete;
  Polygon& operator=(const Polygon&) = delete;
  Polygon(Polygon&&) = delete;
  Polygon& operator=(Polygon&&) = delete;
  // D9 remove value semantics for polymorphic class hierarchies
};

class Triangle final : public Polygon {
// D10 Inheritance should be public
// D11 final prevents unintended extension of hierarchy

public:
  Triangle() = default;
  // D12 Default constructor here finds default constructor in base

  virtual ~Triangle() noexcept = default;
  // D13 virtual even if it's not necessary since class is final
  //     If final is removed, this could prevent a hard to find bug.

  [[nodiscard]] double sumOfAngles() const noexcept override { return 180.0; }
  // D14 Use override. virtual is implied and not required
  // D15 noexcept required by base class

  // NO: Triangle(const Triangle&) = delete;
  // D16 No need to remove value semantics here since it's done in base
};
```

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

Now, before you get too worked up and start yelling at the screen. I know, there are exceptions to almost all of the recommendations in this post. But they are exceptions. What I have presented are the safe and sane defaults. Your starting point for modern C++.

<center>![Hair of fire](hair-on-fire.jpeg)</center>

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>
