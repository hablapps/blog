---
layout: post
title: Macros and Reflective Calls to eliminate boilerplate
date: 2013-03-26 15:32:52.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- case classes
- Embedded DSLs
- immutability
- Macros
- Scala
tags: []
meta:
  _edit_last: '35663564'
  draftfeedback_requests: a:4:{s:32:"habla-computing@googlegroups.com";a:3:{s:3:"key";s:13:"514c47ca8613f";s:4:"time";s:10:"1363953610";s:7:"user_id";s:8:"35663564";}s:36:"juanmanuel.serrano.hidalgo@gmail.com";a:3:{s:3:"key";s:13:"514c4a1da84d3";s:4:"time";s:10:"1363954205";s:7:"user_id";s:8:"35663564";}s:35:"isabel.delamorena.maronas@gmail.com";a:3:{s:3:"key";s:13:"514c4a1df2c22";s:4:"time";s:10:"1363954205";s:7:"user_id";s:8:"35663564";}s:24:"jesus.lopez@hablapps.com";a:3:{s:3:"key";s:13:"514c4a1e46a2d";s:4:"time";s:10:"1363954206";s:7:"user_id";s:8:"35663564";}}
  draft_feedback: "a:2:{s:36:\"juanmanuel.serrano.hidalgo@gmail.com\";a:1:{i:0;a:2:{s:4:\"time\";s:10:\"1364064323\";s:7:\"content\";s:2062:\"\"generate
    boilerplate\": será \"eliminate boilerplate\", ¿no? ;)\n\nyo utilizaría como ejemplo
    de boilerplate las case classes, porque es un ejemplo más realista. Desde luego,
    en scala no es cierto que \"we are forced\" a crear factorías de la forma que
    dices, porque precisamente tenemos las case classes para ello. No es tanto boilerplate
    como el que propones, pero ya es suficiente, ¿no?\n\n<codigo>boilerplate con case
    classes</codigo>\n\nLuego diría que una de las funciones de las case classes es
    proporcionar una factoría a través del companion object, y que eso es lo que vamos
    a tratar de generar automáticamente con macros, de la siguiente forma:\n\n<codigo>sin
    boilerplate con factory</codigo>\n\nAntes de poner el código que genera la macro
    es mejor discutir cómo se podría hacer. Básicamente es el texto que tienes en
    el párrafo siguiente. Y después de la discusión, cuentas la solución:\n\n<codigo>val
    A = new Factory[A]{....} </codigo>\n\n(Pregunta: ¿hace falta el trait Factory
    en este ejemplo? si no queda bien sin esto, justificarlo diciendo que lo vamos
    a completar en siguientes posts ... y casi mejor llamarlo Builder[_] no?)\n\nY
    después la implementación de la macro, que me parece bien.\n\nYo terminaría el
    post hablando de las posibles extensiones: defaults locales (bien como decía Jason
    Zaugg, o utilizando AnnotatedTypes: \"val a1: Int@WithDefault(3)\"), factorías
    parametrizadas, .... \n\nSobre la publicación ahora, o después de publicar Speech:
    yo creo que da igual. Sobre publicarlo así ... yo creo que tendría un poco más
    impacto si pudiéramos incluir alguno de los trabajos futuros. Por ejemplo, si
    pudiéramos incluir las modificaciones a  los defaults podríamos decirle algo al
    Jason. O si resolviéramos los problemas que nos planteó eugenio (no creando los
    builders si se sobreescriben atributos - pero esto es más propio del post sobre
    la siguiente macro, aquí estaría un poco cogido por los pelos), podríamos decirle
    algo a él. Pero bueno, también es un ejemplo interesante de macro, que era la
    idea original :)\";}}s:24:\"jesus.lopez@hablapps.com\";a:1:{i:0;a:2:{s:4:\"time\";s:10:\"1364130799\";s:7:\"content\";s:215:\"
    Hola Jesus, soy Javi. Te ha quedado bastante clara la entrada :) yo no la retocaba.\n\nPD:
    me ha encantado lo de \"In order to avoid lots of dangerous copy-paste actions\".
    Que recuerdos de la antigua arquitectura...xD\";}}}"
  _publicize_pending: '1'
author: Jesus Lopez-Gonzalez
permalink: "/2013/03/26/macros-and-reflective-calls-to-eliminate-boilerplate/"
---
In our previous <a href="http://blog.hablapps.com/2013/03/07/updating-immutable-objects-in-generic-contexts/">post</a>, we told you about <a title="updatable" href="http://github.com/hablapps/updatable">updatable</a>, a library that empowers programmers to build and update immutable objects in generic contexts. We saw the *builder* macro as a main element in the library, but we did not explain in detail how it was implemented. We think it uses an interesting pattern to eliminate boilerplate, so we want to share it with you. Instead of showing the original updatable builder, we are going to use a reduced version, in order to keep the example small. We call it <a href="https://gist.github.com/jesuslopez-gonzalez/5244174#file-factory-scala">factory</a>, because its unique aim is to instantiate traits. Now let's get to work!

Even in Scala, there are situations where we can find annoying boilerplate. That could be the case of the following lines:

```scala
trait A {
  val a1: Int
  val a2: String
}

case class AImpl(a1: Int, a2: String) extends A

AImpl(a1 = 0, a2 = "")

trait B extends A {
  val b1: Double
}

case class BImpl(a1: Int, a2: String, b1: Double) extends B

BImpl(a1 = 3, a2 = "", b1 = 3.0)
```

The case class implements the trait and creates an object factory (among many other things). There is some boilerplate in this implementation that would be nice to eliminate: concretely, the redundant argument list that conforms the constructor. This might be a potential problem if the number of attributes grows excessively. Our approach to eliminate this boilerplate consists on using macros as follows:

```scala
trait A { ... }

val A = builder[A]

A(_a1 = 0, _a2 = "")

trait B extends A { ... }

val B = builder[B]

B(_a1 = 3, _a2 = "", _b1 = 3.0)
```

Scala 2.10 macros are limited in the creation of new types (a limitation that has been lifted with type macros in the macro paradise project), and can only return expressions. So in order to instantiate objects of types A and B we have to exploit anonymous classes. This instances will be returned by the apply method of the object returned by the builder macro.  But you may wonder how is it possible to get these invocations working, since the apply method signature seems to be variable in each case. In fact, there are at least two possible solutions to this problem: either returning a new object of a structural type that declares a custom apply method or simply returning an anonymous function of the proper type. We have chosen here the former alternative in analogy with the way in which the updatable builder is implemented (where you can find additional services besides the factory method). Thus, the code that should be generated by the macro is shown in the next snippet:

```scala
// builder[A]
new {
  def apply(_a1: Int, _a2: String): A = new A {
    val a1 = _a1
    val a2 = _a2
  }
}

// builder[B]
new {
  def apply(_a1: Int, _a2: String, _b1: Double): B = new B {
    val a1 = _a1
    val a2 = _a2
    val b1 = _b1
  }
}
```

Now, it is time for us to analyze the macro implementation. Since we are going to generate dynamic code - mainly in the apply's argument list - it is not feasible to exploit the *reify* macro - which allows the programmer to create the returning expression in a natural way. So, one could choose to use either *parse* or manually create the AST. The latter one is discouraged by the macro author because the code turns pretty verbose. In the current case, if we had used the raw style, the number of lines would have grown remarkably. The reason why this huge growth happens is because creating trait instances produces very complex trees. Nevertheless, we should consider the AST version if we aim to optimize macro execution timings. Next, the macro implementation is shown:

```scala
  def builder[T] = macro builderImpl[T]

  def builderImpl[T: c.WeakTypeTag](c: Context) = {
    import c.universe._
    import c.mirror._

    implicit class SymbolHelper(sym: Symbol) {
      private def cross(t: Type): Type = t match {
        case NullaryMethodType(inner) => inner
        case _ => t
      }
      def name: String = sym.name.encoded
      def tpe: Type = cross(sym.typeSignature)
      def isAccessor: Boolean = sym.isTerm && sym.asTerm.isAccessor
    }

    implicit class TypeHelper(tpe: Type) {
      def name: String = (tpe.typeSymbol: SymbolHelper).name
      def accessors: List[Symbol] = tpe.members.toList.reverse.filter(sym =>
        sym.isAccessor)
    }

    def buildObject = {

      val tpe = weakTypeOf[T]

      def instanceTrait = {
        val vals = tpe.accessors.foldLeft("")(
          (s, sym) => s + s"val ${sym.name} = _${sym.name}\n")
	s"new ${tpe.name} { $vals }"
      }

      def applyArguments = tpe.accessors map { sym =>
	s"_${sym.name}: ${sym.tpe}"
      } mkString ","

      s"""
      new {
	def apply($applyArguments): ${tpe.name} = $instanceTrait
      }
      """
    }

    c.Expr[Object](c.parse(
      s"""
      { val aux = $buildObject; aux } // SI-6992
      """
    ))
  }
```

First, it is important to notice that neither the macro implementation nor the definition declare the result type. Also, note that the parameter type of the *Expr* object returned by the macro is simply *Object.* Thus, we let the compiler infer the proper type of the factory object returned by the macro. Concerning the implementation, we find two main areas in the previous code: reflection tasks and tree creation tasks. The first ones are owned by *SymbolHelper* and *TypeHelper* implicit classes, which extends Symbol and Type, respectively. The *accessor* concept makes reference to the methods that permit the programmer to access the trait values. In the factory's case they have a direct correspondence with the apply's argument list. With respect to the creation tasks, as we said before, the parse method seems to be the better alternative in this case to generate trees. To invoke it, we need a string containing the instructions to reify. That is the *buildFactory*'s duty, which uses string interpolation to format the code that will be finally expanded. This Scala's fresh feature notably improves the instruction's readability.

Currently, we are experimenting with some new ideas to make the factory (and therefore *updatable*) better. Mainly, we would like to generate a parameterized apply method in those situations when the attribute type is abstract:

```scala
trait C {
  type C1
  val c1: C1
}

val C = builder[C]

C[Int](_c1 = 3)
```

We will tell you about this and other extensions in following posts.

To sum up, any programmer knows that boilerplate is not funny at all. In order to avoid lots of dangerous copy-paste actions, external pre-compilation tools may also be employed to solve the issue. However, by doing so, we are adding unnecessary complexity in the project. Today, we have shown how to use macros to face the problem, with a native feature! We have applied it to develop the Speech DSL. How are you planning to use it?

