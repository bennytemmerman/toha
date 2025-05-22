---
title: Bash Variables
weight: 210
menu:
  notes:
    name: Variables
    identifier: notes-bash-variables
    parent: notes-bash
    weight: 10
---
<div style="display: block; width: 100%; max-width: none;">

<!-- Variable -->
{{< note title="Variable" >}}

```bash
NAME="John"
echo $NAME
echo "$NAME"
echo "${NAME}
```

{{< /note >}}

<!-- Condition -->
{{< note title="Condition" >}}

```bash
if [[ -z "$string" ]]; then
  echo "String is empty"
elif [[ -n "$string" ]]; then
  echo "String is not empty"
fi
```

{{< /note >}}

</div>