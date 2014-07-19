---
layout: post
title:  "Migrer vos anciens utilisateurs vers Symfony 2.5"
date:   2014-07-19 13:15:19
categories: symfony2
---

Vous souhaitez migrer vos utilisateurs vers Symfony 2.5 ?
Vous utilisateurs gardent leurs anciens mots de passe, mais si votre ancienne plateforme est un minimum sécurisée vous n'avez qu'un hash de ces mots de passe.
Vous devez donc implémenter la même méthode de hashage que votre ancienne platforme pour authentifier vos anciens utilisateurs.
En parallèle vous souhaitez que vos nouveaux utilisateurs utilisent un autre système de hashage ? Où bien améliorer les système de hashage trop peu sécutisé de votre ancienne plateforme.
Nous allons voir comment la nouvelle version de Symfony (2.5) permet de faire cela.

Cette possibilité n'est pas nouvelle dans la communautée Symfony, en effet le bundle [`FOSAdvancedEncoderBundle`][FOSAdvancedEncoderBundle] propose ces fonctionnalités.
La nouveauté est que Symfony 2.5 intègre maintenant ces possiblités dans le core.

## Ajouter un encodeur de mot de passe

Dans le cas ou vos anciens utilisateurs viennent de drupal 6 vous avez besoin d'ajouter un encoder md5 simple.
Pour cela éditer votre fichier `app/config/security.yml` comme ceci :

{% highlight yml %}
security:
    # …
    encoders:
        # …
        drupal_6_encoder:
            algorithm: md5
            iterations: 1
            encode_as_base64: false
{% endhighlight %}

Le nouvel encodeur `drupal_6_encoder` est maintenant utilisable.

## Utiliser différents encodeurs

Comme vous souhaitez que seul les anciens utilisateurs soit concernés par l'encodeur `drupal_6_encoder` vous devez pour chaques utilisateur préciser l'encodeur à utiliser.
Pour cela votre entité d'utilisateur doit implementer l'interface `EncoderAwareInterface`.

{% highlight php %}
<?php
namespace Acme\UserBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\Encoder\EncoderAwareInterface;

class User implements UserInterface, EncoderAwareInterface
{
    public function getEncoderName()
    {
        if ($this->isDrupal6User()) {
            return 'drupal_6_encoder';
        }

        return null; // utilise l'encoder par defaut
    }

    protected function isDrupal6User() {
        // Votre logique ici
        // Par exemple si vous importez les nid de vos anciens utilisateurs tester la présence de cette propriété
        return !is_null($this->nid);
    }
}
{% endhighlight %}

Retrouvez la [documentation officielle][http://symfony.com/doc/current/cookbook/security/named_encoders.html].

[FOSAdvancedEncoderBundle]:  https://github.com/friendsofsymfony/FOSAdvancedEncoderBundle/blob/master/Resources/doc/index.md

